# EventHub — Booking Management Test Scenarios

Generated: 2026-10-29
Scope: Booking Management — Flow 3 (Book an Event), Flow 4 (View, Cancel, Clear, Refund Eligibility), Flow 6 (Cross-User Security)
Sources: `eventhub-domain` skill (business-rules, user-flows, api-reference), `frontend/app/events/[id]/page.tsx`, `frontend/app/bookings/`, `frontend/components/bookings/BookingCard`, `backend/src/services/bookingService.js`, `backend/src/repositories/bookingRepository.js`, `backend/src/validators/bookingValidator.js`

---

## Happy Path

### TC-001: Book a static event with valid details
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in; a static event (e.g. "Tech Conference Bangalore") has available seats
**Steps**:
1. Navigate to `/events` and click "Book Now" on the event
2. Leave quantity at 1
3. Fill Full Name, Email, Phone Number with valid values
4. Click "Confirm Booking"
**Expected Results**: "Booking Confirmed!" card shows a booking reference (`.booking-ref`), ticket count 1 and total equal to the event price; links to "View My Bookings" and "Browse Events" are shown
**Business Rule**: Flow 3 — Book an Event
**Suggested Layer**: E2E

---

### TC-002: Book multiple tickets and verify total price
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in; event has at least 3 available seats
**Steps**:
1. Open the event detail page
2. Click "+" twice (`#ticket-count` shows 3)
3. Fill valid customer details and confirm
**Expected Results**: Price summary before submit and the confirmation card both show `price × 3`; confirmation shows 3 tickets
**Business Rule**: §9 Price Calculation — `totalPrice = event.price × quantity`
**Suggested Layer**: E2E

---

### TC-003: Book a user-created (dynamic) event
**Category**: Happy Path
**Priority**: P1
**Preconditions**: User is logged in and has created an event via `/admin/events`
**Steps**:
1. Open the user's own event detail page
2. Book 2 tickets with valid details
**Expected Results**: Booking confirmed; event's available seats drop by 2 for this user
**Business Rule**: §6 Per-User Seat Availability (dynamic events)
**Suggested Layer**: E2E

---

### TC-004: View bookings list
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in; user has at least one booking
**Steps**:
1. Navigate to `/bookings`
**Expected Results**: "My Bookings" heading; one `[data-testid="booking-card"]` per booking showing booking reference, booking ID, event title, ticket count, total price, "View Details" and "Cancel" buttons; newest booking first
**Business Rule**: Flow 4 — Manage Bookings; repository orders by `createdAt desc`
**Suggested Layer**: E2E

---

### TC-005: View booking detail page
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in; user has at least one booking
**Steps**:
1. On `/bookings`, click "View Details" on a booking card
**Expected Results**: `/bookings/:id` shows breadcrumb "My Bookings / <ref>", ref badge, "confirmed" status badge, event title, Event Details (event, category, date, venue, city), Customer Details (name, email, phone), Payment Summary (tickets, price per ticket, total paid), Refund section, Booking Information (booked on, `#id`)
**Business Rule**: Booking model fields; Flow 4
**Suggested Layer**: E2E

---

### TC-006: Navigate from confirmation to My Bookings
**Category**: Happy Path
**Priority**: P1
**Preconditions**: User has just completed a booking
**Steps**:
1. Click "View My Bookings" on the confirmation card
**Expected Results**: `/bookings` loads and the new booking is the first card with the same booking reference
**Business Rule**: §1 User Journey
**Suggested Layer**: E2E

---

### TC-007: Cancel a booking from the detail page
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in; user has a booking
**Steps**:
1. Open `/bookings/:id`
2. Click "Cancel Booking"
3. In the dialog "Cancel this booking?", click "Yes, cancel it"
**Expected Results**: Toast "Booking cancelled successfully"; redirected to `/bookings`; booking no longer listed
**Business Rule**: Flow 4 — Cancel booking (delete)
**Suggested Layer**: E2E

---

### TC-008: Cancel a booking from the bookings list
**Category**: Happy Path
**Priority**: P1
**Preconditions**: User has at least 2 bookings
**Steps**:
1. On `/bookings`, click `[data-testid="cancel-booking-btn"]` on one card
2. Confirm in the dialog
**Expected Results**: Only that card is removed; other bookings remain
**Business Rule**: Flow 4 — Cancel booking
**Suggested Layer**: E2E

---

### TC-009: Clear all bookings
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User has 2 or more bookings
**Steps**:
1. On `/bookings`, click "Clear all bookings"
2. Accept the browser confirm "Clear all your bookings? This cannot be undone."
**Expected Results**: All cards removed; "No bookings yet" empty state shown
**Business Rule**: §4 — "Clear All Bookings" removes all bookings in one go
**Suggested Layer**: E2E

---

### TC-010: Check refund eligibility — single ticket
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User has a booking with quantity = 1
**Steps**:
1. Open the booking detail page
2. Click `[data-testid="check-refund-btn"]` ("Check eligibility for refund?")
**Expected Results**: `[data-testid="refund-spinner"]` shows "Checking your refund eligibility…"; then `[data-testid="refund-result"]` shows "Eligible for refund. Single-ticket bookings qualify for a full refund."
**Business Rule**: §8 Refund Eligibility — quantity 1 is eligible
**Suggested Layer**: E2E

---

### TC-011: Retrieve booking by reference via API
**Category**: Happy Path
**Priority**: P2
**Preconditions**: User has a booking with known ref
**Steps**:
1. `GET /api/bookings/ref/<ref>` with the owner's token
**Expected Results**: 200 with the booking, including nested `event`
**Business Rule**: API reference — `GET /api/bookings/ref/:ref`
**Suggested Layer**: API

---

## Business Rules

### TC-100: Booking reference first letter matches event title
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User is logged in
**Steps**:
1. Book "Tech Conference Bangalore"
2. Book "Bollywood Night Mumbai"
**Expected Results**: Refs start with `T-` and `B-` respectively
**Business Rule**: §7 — first letter comes from the event title (uppercase)
**Suggested Layer**: E2E

---

### TC-101: Booking reference format
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User has created a booking
**Steps**:
1. Read the booking ref from the confirmation card or API response
**Expected Results**: Matches `^[A-Z]-[A-Z0-9]{6}$`
**Business Rule**: §7 — `[FIRST_LETTER]-[6_RANDOM_ALPHANUMERIC]`
**Suggested Layer**: API

---

### TC-102: Lowercase event title produces uppercase ref prefix
**Category**: Business Rule
**Priority**: P2
**Preconditions**: User creates an event titled "jazz night"
**Steps**:
1. Book the event
**Expected Results**: Ref starts with `J-`
**Business Rule**: §7 — first letter uppercase
**Suggested Layer**: API

---

### TC-103: Seat count decreases on booking (dynamic event)
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User owns an event with known available seats N
**Steps**:
1. Book 3 tickets
2. Reopen the event detail page
**Expected Results**: Seats shown as `N-3 / totalSeats`
**Business Rule**: §6 — seat count reduces immediately on booking
**Suggested Layer**: E2E

---

### TC-104: Seat count restores on cancellation
**Category**: Business Rule
**Priority**: P0
**Preconditions**: Continue from TC-103
**Steps**:
1. Cancel the 3-ticket booking
2. Reopen the event detail page
**Expected Results**: Seats back to `N / totalSeats`; cancel dialog text said "will release 3 seat(s)"
**Business Rule**: §4 — cancellation immediately frees seats
**Suggested Layer**: E2E

---

### TC-105: Same user can book the same event multiple times
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User is logged in
**Steps**:
1. Book event X twice
**Expected Results**: Two separate bookings with different refs; seats reduced by the sum of both quantities
**Business Rule**: §6 — allows the same user to book the same event multiple times
**Suggested Layer**: API

---

### TC-106: FIFO pruning — 10th booking removes the oldest
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User has cleared all bookings
**Steps**:
1. Create 9 bookings on different events, noting the first booking's ref
2. Create a 10th booking
3. Open `/bookings`
**Expected Results**: Exactly 9 bookings; the first booking is gone; the 10th is present
**Business Rule**: §4 — max 9 bookings, oldest deleted automatically
**Suggested Layer**: API

---

### TC-107: FIFO pruning prefers a booking for a different event
**Category**: Business Rule
**Priority**: P2
**Preconditions**: User has 9 bookings; the oldest is for event A, the second oldest for event B
**Steps**:
1. Create a new booking for event A
**Expected Results**: The oldest booking for a *different* event is pruned (event A's oldest booking remains)
**Business Rule**: Code — `findOldestUserBookingExcludingEvent` used before `findOldestUserBooking`
**Suggested Layer**: API

---

### TC-108: FIFO same-event fallback still reduces seats
**Category**: Business Rule
**Priority**: P2
**Preconditions**: User has 9 bookings, all for the same dynamic event
**Steps**:
1. Book that event again
**Expected Results**: Oldest booking pruned, new booking created, and the event's available seats still drop by the new quantity
**Business Rule**: Code — same-event fallback calls `decrementSeats`
**Suggested Layer**: API

---

### TC-109: Static events are bookable by every user
**Category**: Business Rule
**Priority**: P1
**Preconditions**: Two test accounts (gmail, yahoo)
**Steps**:
1. Each user books the same static event
**Expected Results**: Both bookings succeed; each user sees only their own booking
**Business Rule**: §2 — static events shared across users
**Suggested Layer**: E2E

---

### TC-110: Booking status is always "confirmed"
**Category**: Business Rule
**Priority**: P2
**Preconditions**: User has created a booking
**Steps**:
1. Check booking card / detail badge and API response
**Expected Results**: `status = "confirmed"`; green badge
**Business Rule**: Booking model — status always "confirmed"
**Suggested Layer**: API

---

### TC-111: Total paid matches price × quantity on detail page
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User has a multi-ticket booking
**Steps**:
1. Open booking detail
**Expected Results**: Total Paid = Price per ticket × Tickets
**Business Rule**: §9 Price Calculation
**Suggested Layer**: E2E

---

### TC-112: Refund not eligible for group booking
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User has a booking with quantity = 3
**Steps**:
1. Open booking detail; click "Check eligibility for refund?"
**Expected Results**: After the spinner, result reads "Not eligible for refund. Group bookings (3 tickets) are non-refundable."
**Business Rule**: §8 — quantity > 1 is not eligible
**Suggested Layer**: E2E

---

### TC-113: Booking for an event is removed when the owner deletes the event
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User owns an event and has a booking for it
**Steps**:
1. Delete the event from `/admin/events`
2. Open `/bookings`
**Expected Results**: The booking for the deleted event is gone
**Business Rule**: Booking.eventId cascade delete with Event; Flow 5
**Suggested Layer**: API

---

### TC-114: Filter and paginate the bookings list via API
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User has at least 3 bookings, including bookings for at least 2 events
**Steps**:
1. Call `GET /api/bookings?page=1&limit=2` with the user's token
2. Call `GET /api/bookings?page=2&limit=2` with the same token
3. Call `GET /api/bookings?eventId=<eventId>&status=confirmed` for an event with a booking
**Expected Results**: The first two calls return 2 and 1 bookings respectively with accurate `total`, `page`, `limit`, and `totalPages` metadata; the filtered call contains only confirmed bookings for the requested event
**Business Rule**: API reference — bookings list supports `eventId`, `status`, `page`, and `limit` filters
**Suggested Layer**: API

---

## Security

### TC-200: Cross-user booking detail shows Access Denied
**Category**: Security
**Priority**: P0
**Preconditions**: User A (gmail) has a booking with ID X
**Steps**:
1. Log in as User B (yahoo)
2. Navigate to `/bookings/X`
**Expected Results**: "Access Denied" / "You are not authorized to view this booking."; no booking data shown
**Business Rule**: §2 / Flow 6 — cross-user access returns 403
**Suggested Layer**: E2E

---

### TC-201: Cross-user GET by ID via API
**Category**: Security
**Priority**: P0
**Preconditions**: User A has booking X
**Steps**:
1. `GET /api/bookings/X` with User B's token
**Expected Results**: 403 Forbidden
**Business Rule**: §2 — 403 "Access Denied"
**Suggested Layer**: API

---

### TC-202: Cross-user GET by reference via API
**Category**: Security
**Priority**: P0
**Preconditions**: User A has booking ref R
**Steps**:
1. `GET /api/bookings/ref/R` with User B's token
**Expected Results**: 403 "You do not own this booking"
**Business Rule**: §2 — cross-user access forbidden
**Suggested Layer**: API

---

### TC-203: Cross-user cancel via API does not delete the booking
**Category**: Security
**Priority**: P0
**Preconditions**: User A has booking X
**Steps**:
1. `DELETE /api/bookings/X` with User B's token
2. `GET /api/bookings/X` with User A's token
**Expected Results**: Step 1 is rejected (code returns 404 because lookup is scoped to the caller; docs say 403 — confirm intended code); step 2 still returns the booking
**Business Rule**: §2 — sandbox isolation; code `findById(id, userId)`
**Suggested Layer**: API

---

### TC-204: Clear all only affects the caller's bookings
**Category**: Security
**Priority**: P0
**Preconditions**: Users A and B both have bookings
**Steps**:
1. User A clicks "Clear all bookings"
2. User B opens `/bookings`
**Expected Results**: User B's bookings are unchanged
**Business Rule**: §2 — user sandbox isolation
**Suggested Layer**: API

---

### TC-205: Bookings list never shows other users' bookings
**Category**: Security
**Priority**: P1
**Preconditions**: Users A and B both have bookings
**Steps**:
1. `GET /api/bookings` as User A
**Expected Results**: Every returned booking has User A's `userId`
**Business Rule**: §2 — each user only sees their own bookings
**Suggested Layer**: API

---

### TC-206: Booking endpoints require authentication
**Category**: Security
**Priority**: P0
**Preconditions**: None
**Steps**:
1. Call `GET /api/bookings`, `POST /api/bookings`, `DELETE /api/bookings` and `DELETE /api/bookings/1` with no token, and with an invalid token
**Expected Results**: 401 Unauthorized for every call
**Business Rule**: API reference — Bearer token required
**Suggested Layer**: API

---

### TC-207: Logged-out user opening /bookings
**Category**: Security
**Priority**: P1
**Preconditions**: `localStorage['eventhub_token']` cleared
**Steps**:
1. Navigate to `/bookings`
**Expected Results**: No booking data displayed; user is sent to login (or sees an error state)
**Business Rule**: Auth — client clears token on 401
**Suggested Layer**: E2E

---

### TC-208: Cannot book another user's private event
**Category**: Security
**Priority**: P1
**Preconditions**: User A owns dynamic event E
**Steps**:
1. `POST /api/bookings` with `eventId = E` using User B's token
**Expected Results**: 404 "Event with id E not found"; no booking created
**Business Rule**: §2 — users only see their own dynamic events
**Suggested Layer**: API

---

### TC-209: Customer fields are rendered as text (XSS)
**Category**: Security
**Priority**: P2
**Preconditions**: User is logged in
**Steps**:
1. Book with Full Name `<img src=x onerror=alert(1)>`
2. Open the booking card and detail page
**Expected Results**: Name shown as literal text; no script runs
**Business Rule**: General security — input rendered safely
**Suggested Layer**: E2E

---

## Negative

### TC-300: Name shorter than 2 characters
**Category**: Negative
**Priority**: P1
**Preconditions**: On an event detail page
**Steps**:
1. Enter Full Name "A" with valid email and phone; confirm
**Expected Results**: Inline error "Name must be at least 2 chars"; no API call
**Business Rule**: Booking model — customerName min 2 chars
**Suggested Layer**: Component

---

### TC-301: Invalid email format
**Category**: Negative
**Priority**: P1
**Preconditions**: On an event detail page
**Steps**:
1. Enter email "user@domain"; confirm
**Expected Results**: Inline error "Enter a valid email"
**Business Rule**: customerEmail — valid email format
**Suggested Layer**: Component

---

### TC-302: Phone with fewer than 10 digits
**Category**: Negative
**Priority**: P1
**Preconditions**: On an event detail page
**Steps**:
1. Enter phone "98765"; confirm
**Expected Results**: Inline error "Enter a valid 10-digit phone"
**Business Rule**: customerPhone — min 10 digits
**Suggested Layer**: Component

---

### TC-303: Empty form submitted
**Category**: Negative
**Priority**: P1
**Preconditions**: On an event detail page
**Steps**:
1. Click "Confirm Booking" with all fields empty
**Expected Results**: Errors on name, email and phone; no booking created
**Business Rule**: Required customer fields
**Suggested Layer**: E2E

---

### TC-304: API rejects missing required fields
**Category**: Negative
**Priority**: P1
**Preconditions**: Valid token
**Steps**:
1. `POST /api/bookings` with `{}`
**Expected Results**: 400 "Validation failed" with `details` for eventId, customerName, customerEmail, customerPhone, quantity
**Business Rule**: API — 400 validation error details
**Suggested Layer**: API

---

### TC-305: API rejects quantity outside 1–10
**Category**: Negative
**Priority**: P1
**Preconditions**: Valid token
**Steps**:
1. `POST /api/bookings` with quantity 0, then 11, then 2.5, then "abc"
**Expected Results**: 400 "Quantity must be an integer between 1 and 10" for each
**Business Rule**: Booking model — quantity 1–10
**Suggested Layer**: API

---

### TC-306: Phone with letters rejected by API
**Category**: Negative
**Priority**: P2
**Preconditions**: Valid token
**Steps**:
1. `POST /api/bookings` with customerPhone "98765abcde"
**Expected Results**: 400 "Customer phone must contain only digits and +, -, spaces, or parentheses"
**Business Rule**: Validator — phone character set
**Suggested Layer**: API

---

### TC-307: Booking a non-existent event
**Category**: Negative
**Priority**: P1
**Preconditions**: Valid token
**Steps**:
1. `POST /api/bookings` with `eventId = 999999`
**Expected Results**: 404 "Event with id 999999 not found"
**Business Rule**: Code — event must exist (static or owned)
**Suggested Layer**: API

---

### TC-308: Requesting more seats than available
**Category**: Negative
**Priority**: P0
**Preconditions**: User owns an event with 2 available seats
**Steps**:
1. `POST /api/bookings` with quantity 3
**Expected Results**: 400 "Only 2 seat(s) available, but 3 requested"; no booking created
**Business Rule**: API — 400 Insufficient seats
**Suggested Layer**: API

---

### TC-309: Failed booking at the 9-booking limit must not delete a booking
**Category**: Negative
**Priority**: P1
**Preconditions**: User has 9 bookings
**Steps**:
1. `POST /api/bookings` with a non-existent eventId (or a quantity above available seats)
2. `GET /api/bookings`
**Expected Results**: Request fails (404 / 400) and the user still has 9 bookings. Note: code prunes the oldest booking *before* validating the event and seats — this scenario is expected to expose that defect
**Business Rule**: §4 FIFO pruning should only occur when a new booking is created
**Suggested Layer**: API

---

### TC-310: Cancel a booking that is already cancelled
**Category**: Negative
**Priority**: P2
**Preconditions**: User cancelled booking X
**Steps**:
1. `DELETE /api/bookings/X` again
**Expected Results**: 404 "Booking with id X not found"
**Business Rule**: Code — NotFoundError on missing booking
**Suggested Layer**: API

---

### TC-311: Open a cancelled or non-existent booking detail page
**Category**: Negative
**Priority**: P1
**Preconditions**: User is logged in
**Steps**:
1. Navigate to `/bookings/999999`
**Expected Results**: "Booking not found" — "This booking doesn't exist or may have been cancelled." with "View My Bookings" button
**Business Rule**: UI — non-403 errors show not-found state
**Suggested Layer**: E2E

---

### TC-312: Non-numeric booking ID
**Category**: Negative
**Priority**: P2
**Preconditions**: Valid token
**Steps**:
1. `GET /api/bookings/abc` and `DELETE /api/bookings/abc`
**Expected Results**: 400 or 404 with a clear message — not a 500 server error
**Business Rule**: Robustness — `Number('abc')` is NaN in repository lookups
**Suggested Layer**: API

---

### TC-313: Unknown booking reference
**Category**: Negative
**Priority**: P2
**Preconditions**: Valid token
**Steps**:
1. `GET /api/bookings/ref/Z-000000`
**Expected Results**: 404 `Booking with reference "Z-000000" not found`
**Business Rule**: Code — NotFoundError by ref
**Suggested Layer**: API

---

### TC-314: Dismiss cancel dialog keeps the booking
**Category**: Negative
**Priority**: P2
**Preconditions**: User has a booking
**Steps**:
1. Click "Cancel Booking", then close the dialog without confirming
**Expected Results**: Dialog closes; booking still exists
**Business Rule**: Flow 4 — cancellation requires confirmation
**Suggested Layer**: E2E

---

### TC-315: Dismiss "Clear all bookings" confirm keeps bookings
**Category**: Negative
**Priority**: P2
**Preconditions**: User has bookings
**Steps**:
1. Click "Clear all bookings" and dismiss the browser confirm
**Expected Results**: No API call; all bookings remain
**Business Rule**: UI — `confirm()` guard
**Suggested Layer**: E2E

---

## Edge Cases

### TC-400: Quantity boundaries in the UI (1 and 10)
**Category**: Edge Case
**Priority**: P1
**Preconditions**: Event has at least 10 available seats
**Steps**:
1. Observe "−" at quantity 1
2. Click "+" until it stops
**Expected Results**: "−" disabled at 1; "+" disabled at 10; `#ticket-count` never goes below 1 or above 10
**Business Rule**: Flow 3 — quantity 1–10
**Suggested Layer**: Component

---

### TC-401: Quantity limited by available seats
**Category**: Edge Case
**Priority**: P1
**Preconditions**: User owns an event with 3 available seats
**Steps**:
1. Click "+" repeatedly
**Expected Results**: "+" disabled at 3
**Business Rule**: §6 — per-user seat availability
**Suggested Layer**: Component

---

### TC-402: Book exactly the last available seats
**Category**: Edge Case
**Priority**: P1
**Preconditions**: User owns an event with 2 available seats
**Steps**:
1. Book 2 tickets
2. Reopen the event
**Expected Results**: Booking succeeds; event shows "SOLD OUT" and button reads "Sold Out" (disabled)
**Business Rule**: §6 — seats reach 0
**Suggested Layer**: E2E

---

### TC-403: Exactly 9 bookings — no pruning
**Category**: Edge Case
**Priority**: P1
**Preconditions**: User has 8 bookings
**Steps**:
1. Create the 9th booking
**Expected Results**: All 9 bookings exist; none pruned
**Business Rule**: §4 — limit is 9 (pruning only on the 10th)
**Suggested Layer**: API

---

### TC-404: Name of exactly 2 characters / padded with spaces
**Category**: Edge Case
**Priority**: P2
**Preconditions**: On an event detail page
**Steps**:
1. Book with name "Al" — expect success
2. Book with name " A" (leading space)
**Expected Results**: "Al" accepted. " A" passes the UI check (untrimmed length 2) but the API trims and returns 400 "Customer name must be at least 2 characters" — UI should show that error, not a silent failure
**Business Rule**: customerName min 2 chars; UI/API validation mismatch
**Suggested Layer**: E2E

---

### TC-405: Phone with 10 characters but fewer than 10 digits (API)
**Category**: Edge Case
**Priority**: P2
**Preconditions**: Valid token
**Steps**:
1. `POST /api/bookings` with customerPhone "+91 (98)-7" (10 characters, 5 digits)
**Expected Results**: Should be rejected (rule says min 10 digits). Note: validator checks character length, not digit count — expected to expose a defect
**Business Rule**: Booking model — customerPhone min 10 digits
**Suggested Layer**: API

---

### TC-406: Phone with formatting characters
**Category**: Edge Case
**Priority**: P3
**Preconditions**: On an event detail page
**Steps**:
1. Book with phone "+91 98765 43210"
**Expected Results**: Accepted and displayed as entered on detail page
**Business Rule**: Validator — allows digits, +, -, spaces, parentheses
**Suggested Layer**: E2E

---

### TC-407: Email normalisation
**Category**: Edge Case
**Priority**: P3
**Preconditions**: Valid token
**Steps**:
1. Book with email "Test.User@GMAIL.com"
**Expected Results**: Saved email is the normalised form returned by the API; detail page shows the same value
**Business Rule**: Validator — `normalizeEmail()`
**Suggested Layer**: API

---

### TC-408: Double-click "Confirm Booking"
**Category**: Edge Case
**Priority**: P1
**Preconditions**: On an event detail page with valid form
**Steps**:
1. Double-click "Confirm Booking"
**Expected Results**: Exactly one booking created (button shows loading/disabled while pending)
**Business Rule**: Booking integrity
**Suggested Layer**: E2E

---

### TC-409: Refund check for booking with 10 tickets
**Category**: Edge Case
**Priority**: P3
**Preconditions**: User has a booking with quantity 10
**Steps**:
1. Check refund eligibility
**Expected Results**: "Group bookings (10 tickets) are non-refundable."
**Business Rule**: §8 — quantity > 1 not eligible
**Suggested Layer**: Component

---

### TC-410: Refund result timing (~4 seconds)
**Category**: Edge Case
**Priority**: P2
**Preconditions**: User has any booking
**Steps**:
1. Click "Check eligibility for refund?"
**Expected Results**: Spinner visible immediately; result appears after ~4 s (test with `expect(...).toBeVisible({ timeout: 6000 })`, not `waitForTimeout`)
**Business Rule**: §8 — 4-second spinner
**Suggested Layer**: E2E

---

### TC-411: Cancelling restores seats exactly to the booked quantity
**Category**: Edge Case
**Priority**: P2
**Preconditions**: User owns an event; has two bookings for it (2 and 3 tickets)
**Steps**:
1. Cancel the 2-ticket booking
**Expected Results**: Available seats increase by exactly 2
**Business Rule**: §4 / §6 — seats computed from remaining bookings
**Suggested Layer**: API

---

## UI State

### TC-500: Empty state when user has no bookings
**Category**: UI State
**Priority**: P1
**Preconditions**: User has no bookings
**Steps**:
1. Navigate to `/bookings`
**Expected Results**: "No bookings yet" with description and "Browse Events" button linking to `/events`
**Business Rule**: UI — empty state
**Suggested Layer**: E2E

---

### TC-501: Loading skeletons on bookings page
**Category**: UI State
**Priority**: P3
**Preconditions**: Slow network (throttle or route delay)
**Steps**:
1. Navigate to `/bookings`
**Expected Results**: Booking card skeletons shown until data loads
**Business Rule**: UI — loading state
**Suggested Layer**: Component

---

### TC-502: Error state with Retry
**Category**: UI State
**Priority**: P2
**Preconditions**: `GET /api/bookings` mocked to fail (e.g. 500)
**Steps**:
1. Navigate to `/bookings`
2. Click "Retry" after removing the mock
**Expected Results**: "Couldn't load bookings" / "Failed to connect to the server. Please try again."; Retry reloads the list
**Business Rule**: UI — error state
**Suggested Layer**: E2E

---

### TC-503: "Clearing…" state while clearing bookings
**Category**: UI State
**Priority**: P3
**Preconditions**: User has bookings; clear API delayed
**Steps**:
1. Click "Clear all bookings" and accept
**Expected Results**: Button text "Clearing…" and disabled until the request finishes
**Business Rule**: UI — pending state
**Suggested Layer**: Component

---

### TC-504: Cancel dialog text shows ref and seats
**Category**: UI State
**Priority**: P2
**Preconditions**: User has a 2-ticket booking with ref R
**Steps**:
1. Click Cancel on the card, then on the detail page
**Expected Results**: Card dialog: "This will cancel R and release 2 seat(s) back to the event…"; detail dialog: "Cancelling R will release 2 seat(s) back to the event…"
**Business Rule**: Flow 4 — cancel confirmation
**Suggested Layer**: Component

---

### TC-505: Singular / plural ticket label
**Category**: UI State
**Priority**: P3
**Preconditions**: User has a 1-ticket and a 2-ticket booking
**Steps**:
1. View `/bookings`
**Expected Results**: Cards read "1 ticket" and "2 tickets"; booking form summary reads "× 1 ticket" / "× 2 tickets"
**Business Rule**: UI — pluralisation
**Suggested Layer**: Component

---

### TC-506: Refund button replaced by result
**Category**: UI State
**Priority**: P3
**Preconditions**: User has any booking
**Steps**:
1. Check refund eligibility
**Expected Results**: Button hidden during spinner and after result; result persists until page reload
**Business Rule**: §8 — UI states idle → checking → result
**Suggested Layer**: Component

---

### TC-507: Bookings page shows all 9 bookings on one page
**Category**: UI State
**Priority**: P2
**Preconditions**: User has 9 bookings
**Steps**:
1. Navigate to `/bookings`
**Expected Results**: All 9 cards on page 1; no second page (page size is 10, max bookings 9)
**Business Rule**: §4 — bookings page shows max 9 bookings
**Suggested Layer**: E2E

---

### TC-508: Sandbox limit banner on events page
**Category**: UI State
**Priority**: P3
**Preconditions**: User is logged in
**Steps**:
1. Navigate to `/events`
**Expected Results**: Banner text "Your sandbox holds up to 9 bookings and you can create up to 6 custom events…" displayed per its show/hide condition. Note: business-rules §5 also describes a bookings-page banner, but `frontend/app/bookings/page.tsx` has none — confirm whether it is still required
**Business Rule**: §5 Sandbox Warning Banners
**Suggested Layer**: E2E

---

## Summary

| Category | Range | Count |
|---|---|---|
| Happy Path | TC-001 – TC-011 | 11 |
| Business Rule | TC-100 – TC-114 | 15 |
| Security | TC-200 – TC-209 | 10 |
| Negative | TC-300 – TC-315 | 16 |
| Edge Case | TC-400 – TC-411 | 12 |
| UI State | TC-500 – TC-508 | 9 |
| **Total** | | **73** |

### Likely defects found while reading the code
- **TC-309** — FIFO pruning runs before event/seat validation, so a failed booking at the limit still deletes the user's oldest booking.
- **TC-405** — API validates phone by character length, not digit count.
- **TC-404** — UI checks untrimmed name length; API trims, so " A" passes the UI and fails the API.
- **TC-203** — Cross-user cancel returns 404 while cross-user view returns 403; docs only describe 403.
- **TC-508** — Bookings-page sandbox banner described in docs is not implemented.
