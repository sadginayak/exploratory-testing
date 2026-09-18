# Session-Based Test Charter — Mercedes-Benz Test Drive Booking

## Charter Header
- **Application:** Test Drive Booking (mercedes-benz.co.uk)
- **URL under test:** https://www.mercedes-benz.co.uk/passengercars/mercedes-benz-cars/online-testdrive.html
- **Tester:** Sadgi Nayak
- **Date:** 10.09.2026
- **Time-box:** 90 min
- **Environment:** Production, Chrome,Edge, Windows
- **Testing Approach:** Black-box exploratory testing with network inspection (DevTools)

---

## Mission
**Determine whether the booking system's input validation, error handling, and duplicate prevention are enforced server-side, and whether failures are communicated to the user.**

### Risk
Test Drive booking is a lead-generation funnel. If validation is weak, errors are masked, and duplicate prevention doesn't exist server-side, customers and dealers will experience confusion.

---

## Areas Explored
- [x] Personal Details form field validation (First Name, Last Name, Email, Phone)
- [x] Server-side input validation via HTTP response codes
- [x] Frontend error handling (does UI display server errors?)
- [x] Booking submission with duplicate slot (same dealer, date, time, email)
- [x] Bookable time-slot range vs. dealer's actual operating hours (multiple dealers)
- [ ] Consent checkbox behavior — deprioritized due to time-box
---

## Test Notes (Chronological with Network Evidence)

### Round 1: Basic Input Validation
1. Submitted First Name: `"s"` (single char), Last Name: `"12"` (numeric) → **Status: 400 Bad Request**
2. UI displayed "Thank you for booking" despite 400 error
3. **Finding:** Server rejects invalid data, but frontend masks error

### Round 2: Valid Data Submission
1. Submitted First Name: `"test"`, Last Name: `"tester"` → **Status: 200 OK**
2. Email: `testingnayak1593@gmail.com` → **Status: 200 OK**
3. Received 1 confirmation email
4. **Finding:** Valid data accepted; email confirmation works

### Round 3: Email Format Validation
1. Submitted Email: `test@gmail.c` (single-char TLD) → **Status: 400 Bad Request**
2. UI displayed "Thank you" despite 400 rejection
3. No confirmation email (booking rejected server-side)
4. **Finding:** Server validates email format; frontend hides rejection

### Round 4: Phone Validation
1. Submitted Phone: `+440000000000` (all zeros) → **Status: 200 OK**
2. **Finding:** Server accepts implausible phone numbers (no semantic validation)

### Round 5: Duplicate Prevention Test (Sequential)
1. Booked Dealer "Mercedes-Benz of Temple Fortune" in location London UK, Sept 12, 9:00 with testingnayak1593@gmail.com
2. Repeated same booking immediately from browser Edge with same email ID and slot.
3. Both submissions succeeded
4. Received 2 confirmation emails
5. **Preliminary Finding:** No duplicate prevention (both bookings created)

### Round 6: Duplicate Prevention Test (UI Debounce Check)
1. Clicked Submit button twice rapidly
2. Only 1 API call fired (UI debounce prevented 2nd submission)
3. **Finding:** Frontend has debounce protection, but backend was never tested beacuse it prevented to see the logs.

### Round 7: Duplicate Prevention Test (Simultaneous Requests - Final Verification)
1. **Window 1:** Filled form with Booked Dealer "Mercedes-Benz of Temple Fortune" in location London UK, Sept 12, 12:00 with testingnayak1593@gmail.com
2. **Window 2:** Filled identical form (same email, dealer, date, time) opened in separate window, while in same window the actions happening in one tab were picked by network logs of booking opened in another tab.
3. **Window 1:** Clicked Submit → DevTools Network shows:

```json
PUT /docms/.../upserts Status Code: 200 OK Response: Booking created UI: "Thank you for booking" ✅
```
4. **Window 2:** Clicked Submit (within 1 second) → DevTools Network shows:

```json
PUT /docms/.../upserts Status Code: 400 Bad Request Response: Empty (no error details provided) UI: "Thank you for booking" ❌ ERROR MASKING
```
5. **Finding:** Backend rejected duplicate (400), but frontend displayed success message.

### Round 8: Dealer Hours Mismatch (UI Inspection)
1. Checked dealer operating hours via Google Business
2. Sunday: Google lists closing at 16:00
3. Booking system offered 18:00 Sunday slot
4. Tested across 3 dealers with different listed hours
5. All returned identical 09:00–18:00 slot range
6. **Finding:** Slots are not synced to individual dealer hours

### Round 9: Confirmation Email Content (UX Inspection)
1. Submitted valid booking for Dealer "Mercedes-Benz of Temple Fortune", Mercedes C-Class, Sept 15, 9:00
2. Received confirmation email
3. Email content includes:
   - ✅ Customer name (test)
   - ❌ NO dealer name
   - ❌ NO car model/type
   - ❌ NO booking date/time
   - ❌ NO booking ID/reference number
   - ❌ NO instructions on what to do next
4. **Finding:** Confirmation email lacks essential booking details, customer cannot confirm what they booked
---


## Findings

| # | Finding | Severity | Priority | Evidence | Verification Type | Status |
|---|---------|----------|----------|----------|-------------------|--------|
| 1 | **Frontend Error Masking (CRITICAL)** — Server rejects invalid input (400), but UI shows "Thank you for booking" | CRITICAL | CRITICAL | Network: 400 responses for invalid email/name; UI: "Thank you" shown | Network (DevTools) | Verified ✅ |
| 2 | **Backend Duplicate Prevention Works** — Server rejects identical bookings with 400 status | N/A (not a bug) | N/A | Window 1: 200 OK; Window 2: 400 Bad Request (same data) | Network (DevTools) | Verified ✅ |
| 3 | Phone Field No Semantic Validation — Accepts all-zero numbers (+440000000000) | LOW | MEDIUM | Network: 200 OK for +440000000000 | Network (DevTools) | Verified ✅ |
| 4 | **Bookable Time Slots Don't Match Dealer Hours** — Same 09:00–18:00 for all dealers despite different Google Business hours | HIGH | HIGH | Google Business vs. UI slot display; tested 3+ dealers | UI Inspection + External Data | Verified ✅ |
| 5 | Email Format Validation Works Server-Side | N/A (not a bug) | N/A | Invalid email (s@g.c) → 400; Valid email (testingnayak1593@gmail.com) → 200 | Network (DevTools) | Verified ✅ |
| 6 | Name Format Validation Works Server-Side | N/A (not a bug) | N/A | Numeric names (12) → 400; Text names (test) → 200 | Network (DevTools) | Verified ✅ |
| 7	| Incomplete Confirmation Email — Email contains only customer name; missing dealer, car type, date, time, booking ID	| LOW |	HIGH	| Email inspection: No booking details provided	| UX/Email Content	|Verified ✅

---

## Result:

Server correctly rejects invalid input (400)
Frontend receives 400 but ignores it
Frontend redirects to success page
Customer sees "Thank you" despite booking rejection

### Session Outcome
#### Mission Status: ✅ ACHIEVED

**Recommendation:** 
- **URGENT (Critical/Severity):** Fix frontend error handling; add error message display for 4xx responses
- **HIGH (High/Severity):** Investigate dealer hours data integration; verify slots against real-time dealer schedules
- **HIGH (Priority):** Enhance confirmation email with booking details (dealer, car type, date, time, booking ID)
- **MEDIUM (Priority):** Add phone number semantic validation (reject implausible patterns)

### Verification Strategy Used
1. Network Inspection (DevTools): Examined HTTP status codes, request/response payloads, headers

2. Multi-Window Testing: Tested simultaneous requests to isolate server-side behavior from UI debounce

3. External Data Cross-Reference: Verified dealer hours via Google Business + Mercedes website

4. Payload Analysis: Inspected JSON request bodies to confirm identical data sent in duplicate tests

5. Email Confirmation: Used email receipt as proof of successful booking creation

#### Testing Constraints & Limitations
1. Black-box only: No access to backend code, database, or API documentation

2. CORS blocking: Direct API calls blocked from browser console (by design)

3. Email delays: Some confirmations took time or didn't arrive (rate limiting or async)

4. Production testing: Real bookings created; cannot undo or cleanup

5. UI debounce: Frontend submit button protection prevented testing via normal flow

6. Response body gaps: 400 error had no error message in response (limits diagnostics)

**Conclusion:** 
The Mercedes-Benz Test Drive Booking system has strong server-side validation and duplicate prevention, but critical frontend error handling is broken. Customers cannot trust booking confirmations because the UI masks server errors. This requires immediate remediation.


Charter Status: ✅ COMPLETE