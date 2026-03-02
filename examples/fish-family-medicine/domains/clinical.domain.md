---
domain: Clinical
related_domains:
  - Scheduling
  - Billing
---

CONSTRAINTS:
- This domain does NOT directly access Scheduling data. It only knows about the appointment through the PatientCheckedIn event.
- "Visit" in this context means the clinical encounter, not the appointment slot.
- A Visit cannot exist without a corresponding PatientCheckedIn event. Visits are never created manually.
- Clinical does not handle billing. It emits a VisitCompleted event for Billing to act on.

GLOSSARY:
- Visit: A clinical encounter between a patient and a provider. Created only when a patient checks in. Has notes, diagnosis codes, and a completion status. This is NOT an appointment.
- Chief Complaint: The reason the patient is being seen, in clinical terms. Mapped from the appointment's reason_for_visit but may be refined by the provider.
- Diagnosis Code: An ICD-10 code assigned by the provider. Required before a visit can be marked complete.
- Addendum: A correction or addition to clinical notes after a visit is marked complete. The original notes are preserved.

# ENTITY: Visit
MUST HAVE:
- id: UUID
- status: Enum (InProgress, Complete, Addendum)
- patient_id: UUID
- provider_id: UUID
- appointment_id: UUID
- started_at: DateTime
- completed_at: DateTime (Optional)
- chief_complaint: Text
- notes: Text (Optional)
- diagnosis_codes: List of String (Optional)

RULES:
- R1: A Visit cannot be marked Complete without at least one diagnosis code.
- R2: Notes can be amended after completion (status moves to Addendum) but the original notes are never deleted.
- R3: Only the assigned provider can write notes for a Visit.

RELATIONSHIPS:
- Belongs to Patient
- Belongs to Provider

# LISTENS TO: PatientCheckedIn (from Scheduling domain)
- Creates a new Visit in InProgress status.
- Sets Visit.patient_id, provider_id, and appointment_id from the event payload.
- Sets Visit.chief_complaint from the event's reason_for_visit.

# EVENT: VisitCompleted
TRIGGER:
- When Visit.status changes to Complete.

PAYLOAD:
- visit_id
- patient_id
- provider_id
- diagnosis_codes
- completed_at
