# Event Analytics — Calculation Guide

## Scope

All analytics shown in Event Analytics (`EventAnalytics.jsx`) are **host-scoped** — they reflect data specific to the organizer's own events, not platform-wide metrics.

---

## 1. Attendance & Show-up Quality

| Metric | Formula | Notes |
|--------|---------|-------|
| **Tickets Sold** | Count of tickets with `status ≠ cancelled, refunded` | Number of ticket records (not spots) |
| **Total Attendees** | `Σ ticket.quantity` for active tickets | Sum of spots across all non-cancelled/refunded tickets |
| **Actual Check-ins** | `Σ ticket.spotsCheckedIn` for checked_in + partially_checked_in tickets | Counts individual spots checked in, not tickets |
| **Show-up Rate** | `(checkedIn / totalAttendees) × 100` | Percentage of booked spots that actually showed up |
| **No-Shows** | `totalAttendees - checkedIn` | Spots booked but never checked in |
| **Refunded** | `Σ ticket.quantity` for tickets with `status = refunded` or `refund.status ∈ [requested, processing, processed]` | Counts spots, not tickets |

### Data Source
- All metrics use `Ticket.find({ event: eventId })` — scoped to this specific event only
- `totalAttendees` (aka `totalRegistered`) is the primary denominator — it counts **spots** not **tickets**
- A ticket with `quantity: 4` counts as 4 spots

---

## 2. Demand Timing & Sales Velocity

| Metric | Formula | Notes |
|--------|---------|-------|
| **First Booking Date** | Earliest `purchaseDate` among active tickets | Only counts non-cancelled/refunded tickets |
| **Peak Booking Date** | Last-minute surge detection | Checks if >30% of spots were booked in the last 72 hours before event date |
| **Booking Period** | `ceil((lastBookingDate - firstBookingDate) / 1 day)` | Duration in days from first to last booking |

### Surge Detection Logic
```
eventDate = event.date
last72Hours = eventDate - 72 hours
lastMinuteBookings = Σ quantity of active tickets with purchaseDate ≥ last72Hours
lastMinutePercentage = (lastMinuteBookings / totalAttendees) × 100

if lastMinutePercentage > 30%:
  display "X% bookings in last 72 hrs"
else:
  display "Steady bookings"
```

### Data Source
- Uses only this event's tickets — inherently event-scoped
- Cancelled/refunded tickets are excluded from the surge calculation

---

## 3. Audience Quality Snapshot

### 3a. First-Time Attendees & Repeat Attendees (HOST-SCOPED)

| Metric | Formula | Notes |
|--------|---------|-------|
| **First-Time** | Users with no tickets for any other event by the **same host** | Host-scoped, not platform-wide |
| **Repeat** | Users who have booked at least one other event by the **same host** | Only counts non-cancelled/refunded tickets on other host events |
| **Denominator** | Unique user count (deduplicated) | NOT spots count — each user counts once regardless of ticket quantity |

#### Calculation Steps
1. Get all active tickets for this event → extract unique user IDs
2. Find all events by the same host (`Event.find({ host: hostId })`)
3. Aggregate: find users who have any ticket for another host event (status ≠ cancelled/refunded)
4. Users found in step 3 = repeat; remaining = first-time
5. Percentages = `count / totalUniqueUsers × 100`, rounded to nearest integer

#### Why Host-Scoped?
The organizer dashboard shows analytics for the organizer's business. "Repeat" means a user who has returned to THIS organizer's events — a loyalty metric. Platform-wide repeat data is available in admin analytics.

### 3b. Interest Conversion

| Metric | Formula | Notes |
|--------|---------|-------|
| **Interest Conversion** | `(usersWithMatchingInterests / totalUniqueUsers) × 100` | Event-scoped |

- Checks if user's `interests[]` array contains any of the event's `categories[]`
- Measures how well the event attracted its target audience
- Denominator is unique users, not spots

### 3c. Local Distribution

| Metric | Formula | Notes |
|--------|---------|-------|
| **Local Distribution** | `(localAttendees / totalUniqueUsers) × 100` | Event-scoped |

- Compares user's `location.city` with `event.location.city` (case-insensitive)
- Uses fuzzy matching: `userCity === eventCity || userCity.includes(eventCity) || eventCity.includes(userCity)`
- This handles cases like "Whitefield" matching "Bangalore" area users
- Denominator is unique users, not spots

---

## Summary: What's Event-Scoped vs Host-Scoped

| Section | Scope | Reason |
|---------|-------|--------|
| Attendance & Check-ins | Event-scoped | Only this event's tickets |
| Revenue | Event-scoped | Only this event's ticket revenue |
| Demand Timing | Event-scoped | Only this event's booking dates |
| First-Time / Repeat | **Host-scoped** | Checks across all host's events |
| Interest Conversion | Event-scoped | User interests vs this event's categories |
| Local Distribution | Event-scoped | User city vs this event's city |
| Feedback & Reviews | Event-scoped | Only this event's reviews |

---

## CSV Export Columns

The "Export CSV" button includes two audit columns:
- **City**: User's registered city from their profile (`user.location.city`)
- **First Booked Event**: The first event the user ever booked **on the platform**
