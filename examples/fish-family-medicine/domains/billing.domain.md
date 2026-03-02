---
domain: Billing
related_domains:
  - Clinical
  - Scheduling
---

CONSTRAINTS:
- This domain does NOT directly access Clinical or Scheduling data.
- Billing only learns about visits through the VisitCompleted event.
- Billing only learns about cancellation fees through Scheduling events.
- "Patient" in this context means an account with insurance and payment information, not a clinical record.
- This domain never auto-submits claims. All claims require review before submission.

GLOSSARY:
- Patient: A billing account with insurance information and payment history. Not a clinical record. A patient in Billing may have no visits at all (e.g., pre-registered but never seen).
- Claim: A request for payment submitted to an insurance provider based on a completed visit. Created automatically in Draft status when a visit completes, but never submitted without human review.
- Denial: When an insurance provider rejects a claim. Can be appealed once within 30 days.

# ENTITY: Claim
MUST HAVE:
- id: UUID
- status: Enum (Draft, UnderReview, Submitted, Accepted, Denied, Appealed)
- visit_id: UUID
- patient_id: UUID
- diagnosis_codes: List of String
- amount: Money (Optional, calculated after coding)

RULES:
- R1: A claim cannot be submitted without at least one diagnosis code.
- R2: A claim cannot be submitted more than 90 days after the visit was completed.
- R3: A denied claim can be appealed once within 30 days of denial.

# LISTENS TO: VisitCompleted (from Clinical domain)
- Creates a new Claim in Draft status.
- Sets Claim.visit_id, patient_id, and diagnosis_codes from the event payload.
