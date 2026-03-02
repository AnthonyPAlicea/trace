---
name: Fish Family Medicine
design_system: ../design-system/
---

# Fish Family Medicine

A patient scheduling, clinical visit, and billing system for a small family medicine practice. The system serves three types of users: patients booking and managing their appointments, front desk staff orchestrating the daily schedule, and providers documenting clinical encounters.

## What This System Does

Fish Family Medicine manages the full lifecycle of a patient visit:

1. **Scheduling** - Patients book appointments. Front desk staff manage the daily schedule, handle walk-ins, cancellations, and no-shows.
2. **Clinical** - When a patient checks in, a clinical visit is created. Providers document notes and assign diagnosis codes.
3. **Billing** - When a visit is completed, a billing claim is generated for insurance submission. All claims require human review before submission.

These three domains communicate through events, not direct data access. Scheduling emits a check-in event. Clinical listens and creates a visit. Clinical emits a visit-completed event. Billing listens and creates a claim.

GLOSSARY:
- Patient: A person receiving care at the practice.
- Provider: A doctor, NP, or PA who sees patients.
- Appointment: A scheduled time slot. Not the same as a visit.
- Visit: A clinical encounter that happens when a patient checks in. Not the same as an appointment.
- Claim: A billing request submitted to insurance after a visit is completed.
- Front Desk: Administrative staff who manage the schedule and patient flow.
- Check-In: The moment a patient arrives and is marked present. Bridges Scheduling and Clinical.
