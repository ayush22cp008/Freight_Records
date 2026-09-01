# Chat25 — Node 5 Subnode 5.S2 — Secure Storage Design Decision

## Decision Status
APPROVED — OPTION B

## Decision
The project will NOT approve the minimal bucket-only fix that would leave `/api/upload-photo` as an unauthenticated service-role upload endpoint.

The project approves **Option B — Security Rewrite** for the Check-in photo upload path before enabling the `event-photos` bucket.

## Reason
The storage design investigation verified that the current upload endpoint:
- uses the Supabase service-role key;
- does not authenticate the caller;
- does not bind uploads to a driver/trip identity;
- would therefore preserve a critical security gap if the bucket were simply created.

The project will not knowingly codify this insecure architecture merely to make photo upload work.

## Required Secure Design Direction
Before implementation, the secure design must specify and verify:
1. authenticated driver identity at the upload endpoint;
2. authorization that the driver is permitted to upload evidence for the relevant trip/event;
3. secure trip/driver-bound object path or equivalent binding;
4. appropriate Storage bucket visibility/read-access model compatible with Timeline evidence display;
5. least-privilege upload/read access;
6. cleanup/error behavior when upload succeeds but event insertion fails;
7. exact migration/policy changes required;
8. compatibility with existing Arrival, Check-in, and Departure evidence flows;
9. compatibility with the Node 5 S1 event migration without changing the locked event vocabulary.

## Scope Boundary
This decision does NOT itself authorize implementation.

Do not:
- create the storage bucket yet;
- change Storage policies yet;
- rewrite `/api/upload-photo` yet;
- modify the Node 5 S1 event schema yet;
- change RLS architecture outside the required photo-storage design.

A detailed implementation-ready design must be completed before source/database changes are authorized.

## Relationship to Node 5
Subnode 5.S2 remains separate from the Node 5 S1 event-schema migration. The storage capability is required by the existing evidence-capture flow and must be secured without changing the approved Node 5 event vocabulary.

## Evidence Reference
`03_IMPLEMENTATION/implementation_reports/Chat25_Node5_S2_Checkin_Photo_Storage_Design_Report.md`

## Final State
```text
Option A — bucket only                 REJECTED
Option B — secure upload architecture  APPROVED
Implementation                         NOT YET AUTHORIZED
Storage migration                      NOT YET AUTHORIZED
```
