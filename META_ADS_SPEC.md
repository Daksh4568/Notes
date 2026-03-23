# Meta Ads Integration — How It Works (Simple Overview)

## What Is This?

We've connected **Meta (Facebook/Instagram) Ads** to our website so that when we run paid ads, Meta can track exactly what users do — from clicking an ad to buying a ticket. This helps Meta show our ads to people who are most likely to book events, and tells us which ads and events are making money.

---

## How It Works (The Big Picture)

```
User clicks a Meta Ad
        ↓
Lands on IndulgeOut website (PageView tracked)
        ↓
Browses an event page (ViewContent tracked — which event, city, category)
        ↓
Goes to billing page (InitiateCheckout tracked — amount, quantity)
        ↓
Completes payment via Cashfree
        ↓
Purchase tracked TWICE for accuracy:
  1. Browser sends "Purchase" to Meta  (Frontend Pixel)
  2. Server sends "Purchase" to Meta   (Backend CAPI)
        ↓
Meta matches both, confirms the sale, and optimizes future ad delivery
```

---

## What We Track (4 Events)

| Event | When It Fires | What Data Is Sent | Why It Matters |
|-------|--------------|-------------------|----------------|
| **PageView** | User opens the website | Page URL | Tells Meta how much traffic our ads bring |
| **ViewContent** | User opens an event detail page | Event name, category, city, price, date | Tells Meta which events attract interest |
| **InitiateCheckout** | User clicks "Pay" on billing page | Event ID, amount, quantity, city, category | Shows how many people intended to buy |
| **Purchase** | Payment is confirmed | Amount, ticket count, event name, category, city, user email/phone (hashed) | The main conversion — tells Meta a sale happened |

---

## Two Tracking Systems (Why Both?)

### 1. Frontend Pixel (Browser-Side)
- **What:** A JavaScript snippet (`react-facebook-pixel`) that runs in the user's browser
- **Where:** [frontend/src/utils/metaPixel.js](IndulgeOut/frontend/src/utils/metaPixel.js)
- **How:** When a user views an event or buys a ticket, the browser sends a signal directly to Meta
- **Limitation:** Ad blockers can block it; the browser might close before the signal is sent

### 2. Backend CAPI (Server-Side)
- **What:** Our server sends the Purchase event directly to Meta's API
- **Where:** [backend/utils/metaCapi.js](IndulgeOut/backend/utils/metaCapi.js)
- **How:** When Cashfree sends us a payment webhook, our server calls Meta's Conversions API
- **Advantage:** Cannot be blocked by ad blockers; always fires reliably

### Deduplication
Both systems send the same `orderId` as the event ID. Meta recognizes it's the same purchase and counts it only once — but having both signals improves **Event Match Quality** (Meta's confidence that the conversion is real).

---

## What Data Goes to Meta

### User Data (Hashed for Privacy)
| Data | How It's Sent | Source |
|------|--------------|--------|
| Email | SHA-256 hashed | User's account email |
| Phone | SHA-256 hashed | User's account phone |
| User ID | As `external_id` | Our database user ID |
| IP Address | Plain (required by Meta) | Request headers |
| User Agent | Plain (required by Meta) | Browser string |
| Facebook Browser ID (fbp) | From `_fbp` cookie | Meta Pixel sets this automatically |
| Facebook Click ID (fbc) | From `_fbc` cookie | Set when user clicks a Meta ad |

### Event Data
| Data | Examples |
|------|----------|
| Event name | "Sunset Yoga at Cubbon Park" |
| Event category | "Food & Beverage", "Art, Music & Dance" |
| Event city | "Bangalore", "Mumbai" |
| Event date | "2026-03-25" |
| Ticket price (₹) | 350, 1500 |
| Quantity | 1, 2, 5 |
| Order ID | "order_xyz123" |

---

## Where the Code Lives

### Frontend (3 files)

| File | Purpose |
|------|---------|
| [metaPixel.js](IndulgeOut/frontend/src/utils/metaPixel.js) | Core tracking utility — `initPixel()`, `trackPageView()`, `trackViewContent()`, `trackInitiateCheckout()`, `trackPurchase()` |
| [BillingPage.jsx](IndulgeOut/frontend/src/pages/BillingPage.jsx) | Calls `trackInitiateCheckout()` when user clicks Pay; saves event category/city/date to sessionStorage for the payment callback |
| [PaymentCallback.jsx](IndulgeOut/frontend/src/pages/PaymentCallback.jsx) | After Cashfree redirects back, calls `trackPurchase()` with all data from sessionStorage |

**Additional touchpoints:**
- [App.jsx](IndulgeOut/frontend/src/App.jsx) — calls `initPixel()` + `trackPageView()` on app load
- [EventDetailNew.jsx](IndulgeOut/frontend/src/pages/EventDetailNew.jsx) — calls `trackViewContent()` when event details are fetched

### Backend (2 files)

| File | Purpose |
|------|---------|
| [metaCapi.js](IndulgeOut/backend/utils/metaCapi.js) | Server-side API call to Meta's Conversions API — `sendPurchaseEvent()` and `sendInitiateCheckoutEvent()` |
| [payments.js](IndulgeOut/backend/routes/payments.js) | Calls `sendPurchaseEvent()` inside the Cashfree webhook handler after confirming payment |

### Configuration
- **Pixel ID:** `1462994898219509` (hardcoded in both metaPixel.js and metaCapi.js)
- **Access Token:** Stored as `META_ACCESS_TOKEN` environment variable (backend only)
- **Frontend URL:** `FRONTEND_URL` env var (used for `event_source_url` in CAPI)

---

## Event Match Quality Score

This is Meta's confidence score (1-10) that a conversion on our website matches back to a real Meta user. Higher score = better ad optimization.

**What we send that improves the score:**

| Parameter | Impact | Status |
|-----------|--------|--------|
| Email (hashed) | High | ✅ Sending |
| Phone (hashed) | High (+11%) | ✅ Sending |
| IP Address | Medium | ✅ Sending |
| User Agent | Medium | ✅ Sending |
| Facebook Browser ID (fbp) | High (+22%) | ✅ Sending from browser |
| Facebook Click ID (fbc) | Very High (+100%) | ✅ Sending from browser |
| External ID (User ID) | High (+22%) | ✅ Sending |

**Current estimated score:** 8.0-9.0 out of 10

---

## What You Can See in Meta Ads Manager

With this integration, you can:

1. **See which events get the most views** — breakdown by Content ID
2. **See which event categories convert best** — e.g. "Food & Beverage" has 4.2x ROAS vs "Sports" at 2.9x
3. **See which cities perform best** — e.g. Bangalore converts better than Delhi
4. **Track the full funnel** — how many people View → Checkout → Purchase
5. **Calculate ROAS** — for each ad, campaign, event, category, and city
6. **Create targeted audiences** — retarget event viewers, exclude past purchasers

---

## Possible Future Optimizations

### High Impact (Recommended Next)

| Optimization | What It Does | Effort |
|---|---|---|
| **Pass fbp/fbc to backend** | Store Facebook cookie IDs in sessionStorage and send them to the server via the payment API so CAPI also has them — boosts server-side Match Quality from ~6 to ~9 | Small |
| **Track Search events** | Fire `ReactPixel.track('Search', { search_string: query })` when users search on Explore page — helps Meta understand user intent | Small |
| **Track AddToCart** | Fire when user selects ticket quantity on billing page — adds another funnel step for better optimization | Small |
| **Track CompleteRegistration** | Fire when a new user signs up — helps build signup audiences and optimize signup campaigns | Small |

### Medium Impact

| Optimization | What It Does | Effort |
|---|---|---|
| **Dynamic Product Ads (Catalog)** | Upload events as a product catalog to Meta — enables automatic retargeting ads showing the exact events users viewed | Medium |
| **Offline Conversions** | After events happen, upload check-in data to Meta — tells Meta which ticket buyers actually attended, improving audience quality | Medium |
| **Value-Based Lookalikes** | Create lookalike audiences from high-value purchasers (bought ₹1000+ tickets) — targets people similar to your best customers | Small (config-only) |
| **Server-side InitiateCheckout** | Currently only browser-side; adding CAPI for checkout would improve tracking reliability | Small |

### Low Priority (Nice to Have)

| Optimization | What It Does | Effort |
|---|---|---|
| **Track Share events** | Fire when users share an event — measures organic reach from paid traffic | Small |
| **Track WishlistAdd** | If you add a "Save for later" feature — signals strong intent to Meta | Small |
| **Multi-Pixel support** | If you ever run ads for organizers/brands separately — each gets their own pixel | Medium |
| **Conversion API Gateway** | Use Meta's hosted CAPIG instead of self-hosted — removes server load and improves reliability | Medium |

---

## Safety & Privacy

- **User data is hashed** — emails and phones are SHA-256 hashed before sending to Meta. Meta cannot reverse the hash.
- **No PII in custom data** — event names, categories, and cities are not personally identifiable.
- **Graceful failure** — if Meta tracking fails at any point, it never breaks the payment flow or user experience. All tracking calls are wrapped in try-catch.
- **Ad blocker resilient** — the backend CAPI fires regardless of user's browser extensions.
- **GDPR note** — you should have a cookie consent banner if targeting EU users. Currently, the Pixel fires on every page load.

---

## Quick Reference

```
Frontend Pixel ID:  1462994898219509
Backend Pixel ID:   1462994898219509  (same)
Backend Access Token: META_ACCESS_TOKEN env var
Graph API Version:  v18.0
CAPI Endpoint:      https://graph.facebook.com/v18.0/{PIXEL_ID}/events
```
