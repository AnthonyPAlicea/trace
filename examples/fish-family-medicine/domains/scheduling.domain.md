---
domain: Scheduling
related_domains:
  - Clinical
---

CONSTRAINTS:
- Scheduling owns appointment lifecycle only. It does not know about clinical notes, diagnoses, or billing.
- "Appointment" in this context is a time slot. It is NOT a visit.
- When a patient checks in, Scheduling emits an event. What happens clinically after that is not Scheduling's concern.
- Never include reason_for_visit in SMS messages (PHI constraint).

EXTERNAL: Twilio SMS

GLOSSARY:
- Appointment: A scheduled time slot for a patient to see a provider. Not a visit. An appointment can be cancelled or no-showed without a visit ever existing.
- Check-In: The moment a patient arrives and is marked present. This ends Scheduling's responsibility and triggers the Clinical domain.
- Provider: A doctor, NP, or PA who sees patients. Same meaning as global glossary.
- No-Show: A patient who had a confirmed appointment but never checked in.

# ENTITY: Appointment
MUST HAVE:
- id: UUID
- status: Enum (Requested, Confirmed, CheckedIn, Cancelled, NoShow)
- appointment_time: DateTime
- duration_minutes: Integer
- reason_for_visit: Text
- patient_id: UUID
- provider_id: UUID

RULES:
- R1: Status moves forward only: Requested -> Confirmed -> CheckedIn. Cancelled and NoShow can happen from Requested or Confirmed.
- R2: A provider cannot have overlapping appointments.
- R3: A new patient appointment must be at least 30 minutes. Returning patient appointments can be 15 minutes.
- R4: Appointments cannot be scheduled more than 6 months in advance.
- R5 STUB: Cancellations less than 24 hours before appointment time incur a cancellation fee.

RELATIONSHIPS:
- Belongs to Patient
- Belongs to Provider

# ENTITY: Provider
MUST HAVE:
- id: UUID
- name: String
- specialty: String
- accepting_new_patients: Boolean

# EVENT: PatientCheckedIn
TRIGGER:
- When Appointment.status changes to CheckedIn.

PAYLOAD:
- appointment_id
- patient_id
- provider_id
- reason_for_visit

# EVENT: AppointmentConfirmed
TRIGGER:
- When Appointment.status changes to Confirmed.
- Send a confirmation text via Twilio SMS to the Patient's phone number with the appointment date, time, and provider name.

PAYLOAD:
- appointment_id
- patient_id
- appointment_time
- provider_name

# LISTENS TO: Twilio SMS
- When a patient replies CANCEL, find the matching appointment by patient phone number and cancel it. Appointment.R1 and Appointment.R5 still apply.
