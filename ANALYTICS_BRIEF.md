# IndulgeOut — Analytics & User Intelligence Brief

## Table of Contents
1. [Overview](#overview)
2. [Current Analytics Data — What We Save](#current-analytics-data)
3. [B2C User Analytics](#b2c-user-analytics)
4. [B2B Analytics — Community Organizers](#b2b-community-organizers)
5. [B2B Analytics — Brands / Sponsors](#b2b-brands--sponsors)
6. [B2B Analytics — Venues](#b2b-venues)
7. [Platform-Wide Tracking](#platform-wide-tracking)
8. [Recommendation Engine (Current)](#recommendation-engine)
9. [Optimization Opportunities](#optimization-opportunities)

---

## Overview

IndulgeOut tracks user activity across **4 user roles** (B2C users, Community Organizers, Brands, Venues) and **1 admin role**. Analytics data feeds into personalized recommendations, targeted notifications, admin dashboards, and revenue reporting.

---

## Current Analytics Data

### What We Currently Save in the Database

#### Per-User Analytics (`User.analytics`)

| Data Point | Fields Stored | Purpose |
|------------|--------------|---------|
| **Event Registrations** | event (ref), category, registeredAt, attended (bool), rating (1-5), location (city/state) | Track which events users attend, by category & location |
| **Community Participation** | community (ref), category, joinedAt, activityLevel (low/medium/high), postsCount, likesReceived | Measure community engagement depth |
| **Location History** | city, state, country, coordinates, frequency, lastSeen | Understand geographic behavior patterns |
| **Category Preferences** | category, score (weighted), lastInteraction | Weighted scoring: register=2pts, click=1pt — decays over time |
| **Search History** | query, filters (category/location/price), resultsCount, clickedEvent, searchedAt | Capture search intent & conversion |
| **Recommendation Metrics** | totalShown, clicked, registered, CTR, conversionRate | Measure recommendation quality |

#### User Preferences

| Field | Values | Purpose |
|-------|--------|---------|
| Notification toggles | email, push, event reminders, community updates | Channel preference |
| Recommendation frequency | daily / weekly / monthly | Engagement cadence |
| Max travel distance | km (default 50) | Geo-targeting radius |
| Preferred event times | morning / afternoon / evening / night | Time-based targeting |
| Price range | min–max (default ₹0–₹10,000) | Budget-based filtering |

---

## B2C User Analytics

### What We Track

| Activity | Data Saved | Notification Trigger |
|----------|-----------|---------------------|
| **Event Registration** | category, location, timestamp, ticket type, payment amount | Similar events in same category & city |
| **Event Attendance** | check-in time, check-in method (QR scan) | Post-event rating request |
| **Event Rating** | 1-5 stars, comment, photos, verified attendee flag | Milestone: first review |
| **Event Saves** | savedEvents[] array on User model | Reminder before saved event date |
| **Search Behavior** | query text, filters used, results seen, event clicked | Trending searches → popular events |
| **Category Clicks** | weighted score per category (click=1, register=2) | Top category → push similar events |
| **Location Movement** | city frequency, last seen, coordinates | Events near user's frequent locations |
| **Ticket Purchases** | basePrice, GST, platformFees, coupon used, discount | Price-sensitive recommendations |
| **Referrals** | referral code, count, credits earned | Referral milestone notifications |
| **Community Joins** | community ref, category, joinedAt | Community event notifications |

### Current Notification Triggers for B2C

- **Event reminders**: 24h before registered event
- **Post-event rating**: Day after event ends
- **Similar event discovery**: Based on category preferences + location
- **Saved event reminders**: Before saved event dates
- **New event in category**: When new events match top category preferences
- **Community updates**: Activity from joined communities

---

## B2B — Community Organizers

### Profile Data Tracked

| Data | Fields | Analytics Value |
|------|--------|----------------|
| **Community Profile** | communityName, city, category[], communityType (open/curated), description | Segmentation & matching |
| **Past Experience** | pastEventExperience (0-5 to 100+), typicalAudienceSize | Credibility scoring |
| **Hosting Preferences** | preferredCities[], preferredCategories[], preferredEventFormats[], preferredAudienceTypes[] | Collaboration matching |
| **Social Links** | instagram, facebook, website, linkedin | Reach estimation |

### Event Performance Tracked

| Metric | How Tracked | Purpose |
|--------|------------|---------|
| **Event Views** | analytics.views, uniqueViews, viewHistory[] (daily) | Demand measurement |
| **Click-Through Rate** | analytics.clicks / views | Content quality signal |
| **Registration Conversion** | analytics.registrations / views | Funnel optimization |
| **Attendance Rate** | checked_in tickets / total tickets | Actual delivery vs. promise |
| **Revenue per Event** | ticket sales × price tiers, coupon usage | Financial performance |
| **Average Rating** | avgRating from Review model | Quality signal |
| **Repeat Attendees** | Cross-event participant overlap | Community loyalty indicator |
| **Capacity Utilization** | registrations / maxParticipants | Pricing strategy signal |

### Current Notifications for Organizers

- **Low booking alert**: <40% capacity with event <7 days away
- **Draft event reminder**: Unpublished events older than 7 days
- **Profile incomplete**: Missing required profile fields
- **KYC pending**: Missing payout/bank details
- **Collaboration proposals**: New B2B partnership requests
- **Monthly settlement report**: Revenue summary (1st of month)

---

## B2B — Brands / Sponsors

### Profile Data Tracked

| Data | Fields | Analytics Value |
|------|--------|----------------|
| **Brand Profile** | brandName, brandCategory, targetCity[], description | Targeting |
| **Sponsorship Types** | barter, paid_monetary, product_sampling, co-marketing | Collaboration matching |
| **Collaboration Intent** | sponsorship, sampling, popups, experience_partnerships, brand_activation, content_creation | Intent-based matching |
| **Budget Range** | min–max (INR) | Deal sizing |
| **Hosting Preferences** | preferredCategories[], preferredEventFormats[], preferredCollaborationTypes[], preferredAudienceTypes[] | Auto-matching |

### Collaboration Tracking

| Metric | Data | Purpose |
|--------|------|---------|
| **Proposals Sent/Received** | count, status progression | Pipeline health |
| **Counter-Proposals** | modified fields, pricing changes | Negotiation patterns |
| **Workspace Activity** | field edits, forum messages, agreement progress | Engagement quality |
| **Collaboration Completion Rate** | confirmed / total proposals | Reliability scoring |
| **Response Time** | viewedAt - createdAt | Responsiveness signal |

---

## B2B — Venues

### Profile Data Tracked

| Data | Fields | Analytics Value |
|------|--------|----------------|
| **Venue Profile** | venueName, city, venueType, capacity, amenities[], operatingDays/Hours | Inventory |
| **Pricing** | rentalFee, coverChargePerGuest, revenueSharePercentage, commercialModel | Revenue optimization |
| **Rules & Restrictions** | alcohol, smoking, ageLimit, soundCutoff, F&B exclusivity | Event compatibility filter |
| **Space Details** | spaceType (Indoor/Outdoor/Rooftop), seatingCapacity, parking | Logistics matching |
| **Hosting Preferences** | preferredCategories[], preferredEventFormats[], preferredAudienceTypes[] | Auto-matching |

### Venue Performance Tracked

| Metric | Source | Purpose |
|--------|--------|---------|
| **Events Hosted** | Events with venue as host | Utilization rate |
| **Revenue Generated** | Ticket sales from venue events | Financial performance |
| **Average Attendance** | Participant counts across events | Demand signal |
| **Repeat Organizer Rate** | Same organizer booking multiple times | Satisfaction indicator |

---

## Platform-Wide Tracking

### Payment & Settlement Pipeline

| Stage | Status | Data Saved |
|-------|--------|-----------|
| **Purchase** | pending | orderId, basePrice, GST, platformFees, couponDiscount |
| **Reconciliation** | verified / mismatch / manual_review | Cashfree order verification, amount match |
| **Settlement** | captured → settled | UTR, settlementAmount, cashfreeServiceCharge, serviceTax, settlementDate |
| **Payout** | pending → completed | Organizer payout with event breakdown |

### Admin Dashboard Analytics

- Platform-wide user counts by role
- Event counts (total, active, live)
- Collaboration pipeline (pending, confirmed, rejected, completed)
- Revenue aggregates with growth percentages (30-day vs. prior 30-day)
- Per-organizer performance (events, revenue, ratings)
- Reconciliation health (mismatches, failed verifications)

---

## Recommendation Engine

### Current Algorithm (Weighted Scoring)

| Factor | Weight | Input Data |
|--------|--------|-----------|
| **Interest Matching** | 40% | user.interests + categoryPreferences vs. event.categories |
| **Location Proximity** | 25% | user.location + locationHistory vs. event.location |
| **Behavioral Patterns** | 20% | Past registrations by category, time preferences, recency |
| **Community/Social** | 15% | Events from user's joined communities |
| **Temporal Boost** | 10% | Weekend/evening events get slight boost |
| **Popularity** | 5% | Registration velocity, click count |

### Current Tracking Methods

- `trackRecommendationInteraction()` — updates CTR, conversion rate
- `updateCategoryPreference()` — adjusts weighted category scores
- `updateLocationHistory()` — tracks geographic frequency patterns

---

## Optimization Opportunities

### 1. B2C User Targeting Enhancements

| Enhancement | What to Save | How It Helps |
|-------------|-------------|-------------|
| **Session Duration** | Time spent on event pages, scroll depth | Distinguish casual browsers from high-intent users |
| **Abandoned Registrations** | Events where user started but didn't complete | Retargeting via push/email nudge |
| **Social Sharing** | Which events users share, to which platform | Viral coefficient per event type |
| **Time-of-Day Activity** | Login/browse timestamps (hourly buckets) | Optimize notification send times |
| **Price Sensitivity Score** | Track coupon redemption rate, free vs. paid ratio | Smart pricing recommendations |
| **Friend/Group Attendance** | Who attends events together (group bookings) | "Your friends are going" notifications |
| **Event Completion Rate** | Check-in time vs. event end time (did they stay?) | True satisfaction signal beyond rating |
| **Post-Event Behavior** | Did they follow organizer, save similar events, share photos? | Organic advocacy tracking |
| **Churn Risk Score** | Days since last registration, declining login frequency | Win-back campaigns before user goes inactive |
| **Lifetime Value (LTV)** | Cumulative spend, frequency, avg ticket price trend | VIP tier identification |

### 2. B2B Community Organizer Enhancements

| Enhancement | What to Save | How It Helps |
|-------------|-------------|-------------|
| **Audience Demographics** | Age/gender distribution of event attendees | Help brands find right audience |
| **Repeat Attendee %** | % of attendees who've attended organizer's past events | Community loyalty metric for sponsors |
| **Cross-Category Performance** | Revenue/attendance by event category | Guide organizers to their strongest categories |
| **Optimal Pricing Analysis** | Price point vs. conversion rate across events | Dynamic pricing recommendations |
| **Day-of-Week Performance** | Attendance by day of week | Best day recommendations |
| **Marketing Channel Attribution** | Where did registrations come from (platform, social, link) | ROI per marketing channel |
| **Collaboration Success Score** | Completed collabs / total proposals, avg negotiation time | Trust signal for B2B matching |
| **Cancellation Rate** | Events cancelled or significantly changed post-publish | Reliability metric |

### 3. B2B Brand/Sponsor Enhancements

| Enhancement | What to Save | How It Helps |
|-------------|-------------|-------------|
| **Brand Exposure Metrics** | Events sponsored, total audience reached, impressions estimate | ROI reporting for brand managers |
| **Audience Match Score** | % overlap between brand's target audience and event attendees | Smart matching for proposals |
| **Collaboration ROI Tracker** | Pre/post metrics: brand mentions, social follows, sampling uptake | Justify sponsorship spend |
| **Preferred Deal Structure** | Historical negotiation patterns (accepts cash? prefers barter?) | Speed up future negotiations |
| **Competitor Activity** | Which brands sponsor similar category events | Market intelligence |
| **Campaign Calendar** | Past/upcoming sponsorship timeline | Seasonal planning |
| **Response Rate & Time** | How quickly brand responds to proposals | Reliability signal for organizers |

### 4. B2B Venue Enhancements

| Enhancement | What to Save | How It Helps |
|-------------|-------------|-------------|
| **Utilization Heatmap** | Bookings by day-of-week and time slot | Identify idle capacity, offer dynamic pricing |
| **Event-Type Performance** | Which event categories perform best at venue | Attract right organizers |
| **Revenue per Sq Ft** | Normalize revenue by venue size | Benchmark against similar venues |
| **Repeat Booking Rate** | Same organizer rebooking | Venue satisfaction signal |
| **Seasonal Trends** | Monthly booking volume and revenue | Anticipate demand dips |
| **Amenity Utilization** | Which amenities are most requested/used | Investment prioritization |
| **Review Sentiment** | NLP-parsed themes from event reviews at venue | Actionable feedback |

### 5. Platform-Wide Intelligence

| Enhancement | What to Save | How It Helps |
|-------------|-------------|-------------|
| **Cohort Analysis** | Group users by signup month, track retention curves | Measure product-market fit |
| **Funnel Analytics** | Browse → Save → Register → Attend → Review → Repeat | Identify drop-off points |
| **A/B Test Framework** | Feature flags with conversion tracking per variant | Data-driven product decisions |
| **City-Level Health Score** | Active users, events, revenue per city | Expansion strategy |
| **Cross-Sell Tracking** | Users who attend events AND browse sponsors/venues | B2C → B2B conversion path |
| **Notification Engagement** | Open rate, click rate, dismiss rate per notification type | Optimize notification strategy |
| **Content Performance** | Which event descriptions, photos, titles get highest CTR | Content guidelines for organizers |
| **Fraud Signals** | Bulk registrations, rapid cancellations, suspicious patterns | Platform safety |

### 6. Advanced Recommendation Improvements

| Enhancement | Implementation | Impact |
|-------------|---------------|--------|
| **Collaborative Filtering** | "Users like you also attended…" using cosine similarity on registration vectors | Better cold-start recommendations |
| **Time-Decay Weighting** | Recent activity weighted higher (exponential decay, half-life: 30 days) | Recommendations reflect current interests, not stale data |
| **Geographic Clustering** | Cluster users by frequent locations, surface events in all frequent areas | Don't limit to city — include travel patterns |
| **Price Elasticity Model** | Track conversion at different price points per user | Personalized pricing nudges |
| **Event Sequence Patterns** | What do users attend after a workshop? After a mixer? | Next-event prediction |
| **Social Graph** | Build implicit connections from co-attendance | "Your connections are attending" |
| **Real-Time Trending** | Events gaining registrations fastest in user's area | FOMO-driven conversion |
| **Weather-Aware** | Boost indoor events when rain forecasted | Context-aware recommendations |

---

## Summary

### Currently Saving (Strong Foundation)
- Event registration, attendance, ratings per user
- Category preferences with weighted scoring
- Location history with frequency tracking
- Search behavior with click-through tracking
- Full payment lifecycle (purchase → reconciliation → settlement)
- Recommendation interactions (shown, clicked, registered)
- B2B collaboration pipeline with workspace activity logs
- Community engagement metrics (posts, likes, activity level)

### Top Priority Optimizations to Add
1. **Abandoned registration tracking** — highest-impact recovery opportunity
2. **Notification engagement metrics** — optimize send times and channel mix
3. **Audience demographics on organizer profiles** — key data for brand sponsors
4. **Collaborative filtering** — dramatically improves recommendation quality
5. **Churn risk scoring** — proactive retention before users go inactive
6. **City-level health scoring** — data-driven expansion decisions
7. **Brand exposure / ROI reporting** — justifies sponsor spend, drives repeat collaborations
