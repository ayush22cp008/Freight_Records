# Chat50 — Day21 — Node 7
## Same-Company Sender/Receiver Governance — Ayush Manual Verification Test Result

**Status:** 🟢 VERIFIED / PASS  
**Verifier:** Ayush  
**Verification type:** Deployed UI + direct authenticated API manual verification  
**Scope:** NEW Trip creation only

---

## 1. Verification Objective

Verify that a Company cannot create a NEW Trip where the Sending Company and Receiving Company are the same company, while normal cross-company Trip creation remains the intended valid path.

Canonical invariant:

```text
Trip.company_id != Trip.receiving_company_id
```

---

## 2. UI Verification

Authenticated Company account used:

```text
Company: testc2
Company ID: 567c472b-7763-4284-a78c-e12fd9eb0078
```

On the deployed Create Trip page, the Receiving Company selector was opened and inspected.

Observed:

```text
Own company (testc2) available as receiver  → NO
Other companies available                  → YES
```

Classification:

```text
🟢 PASS
```

---

## 3. Direct API Bypass Verification

A direct authenticated `POST /api/trips/create` request was sent from the logged-in `testc2` browser session with:

```text
receiving_company_id = 567c472b-7763-4284-a78c-e12fd9eb0078
```

The request intentionally bypassed the UI selector to test server-side enforcement.

Observed response:

```text
HTTP STATUS: 400
error: "Sender and receiving company cannot be the same for new trips"
```

Classification:

```text
🟢 PASS
```

The server therefore rejects the prohibited A → A creation attempt.

---

## 4. Persistence / Post-Rejection Verification

After the rejected API request, Ayush opened **My Created Trips** for `testc2`.

Observed:

```text
"You have no active created trips at this time."
```

The test Trip label used in the rejected request was:

```text
CHAT50 TEST SAME COMPANY
```

It was not present in the visible created-trip list.

Classification:

```text
🟢 PASS
```

This is consistent with the API returning HTTP 400 before successful Trip creation. No direct database query was performed during this manual test.

---

## 5. Manual Verification Matrix

| Test | Expected | Observed | Result |
|---|---|---|---|
| Own company appears in Receiving Company selector | Must not appear | `testc2` absent | 🟢 PASS |
| Direct API A → A attempt | Must return 400 | HTTP 400 | 🟢 PASS |
| API rejection message | Same-company creation rejected | Correct rejection message | 🟢 PASS |
| Test Trip after rejection | Must not be created | No test Trip visible | 🟢 PASS |

---

## 6. Final Ayush Decision

```text
NEW same-company Sender/Receiver rule
→ 🟢 AYUSH MANUAL VERIFICATION COMPLETE / PASS
```

The Chat50 implementation is manually verified at the deployed application level for the tested `testc2 → testc2` case.

Existing legacy same-company Trips were not deleted, reassigned, or rewritten by this manual test.

---

## 7. Next Project Gate

The same-company governance implementation is complete and manually verified.

Proceed to the next Node 7 checkpoint:

```text
Cross-Portal End-to-End validation
→ next
```
