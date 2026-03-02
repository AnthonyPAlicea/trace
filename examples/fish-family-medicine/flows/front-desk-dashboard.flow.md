---
flow: FrontDeskDashboard
strategy: denormalize
persona: FrontDesk
domains:
  - Scheduling
---

# SCREEN: Today's Schedule
> GOAL: See everything at a glance. Manage the day's flow without navigating away.

## FRAME: Schedule View
- REF: design-system/templates/data-table-with-status.jsx
- REF: design-system/examples/schedule-grid.png
- SOURCE: Appointment entity (filtered by date == today, ordered by appointment_time)
- Shows: Patient name, appointment time, duration, reason, status.
- Color-code by status: Confirmed (neutral), CheckedIn (green), NoShow (red), Cancelled (gray).
- ACTION: Click appointment -> Transitions to FRAME: AppointmentDetail
- ACTION: "Add Walk-In" -> Transitions to FRAME: QuickAdd

## FRAME: AppointmentDetail
- REF: design-system/components/detail-sidebar.jsx
- REF: design-system/mockups/appointment-sidebar.png
- ENTERS_FROM: FRAME: Schedule View

Slides in as a sidebar over the schedule grid. The front desk clerk needs to see the appointment context and take action without losing sight of the rest of the day. Insurance status should be visually prominent since it affects check-in.

- DATA:
  - Appointment.status
  - Appointment.appointment_time
  - Appointment.reason_for_visit
  - Patient.name
  - Patient.insurance_status
- ACTION: "Check In" -> Changes Appointment.status to CheckedIn, Transitions to FRAME: Schedule View. Only available if status is Confirmed.
- ACTION: "Mark No-Show" -> Changes Appointment.status to NoShow, Transitions to FRAME: Schedule View. Only available if appointment_time has passed and status is Confirmed.
- ACTION: "Cancel" -> Transitions to FRAME: CancelConfirm
- ACTION: "Reschedule" -> Transitions to FRAME: Reschedule
- ACTION: "Back" -> Transitions to FRAME: Schedule View

## FRAME: CancelConfirm
- REF: design-system/components/confirmation-dialog.jsx
- ENTERS_FROM: FRAME: AppointmentDetail
- DATA: Appointment details summary.
- Shows cancellation fee warning if less than 24 hours before appointment (Appointment.R5).
- ACTION: "Confirm Cancellation" -> Changes Appointment.status to Cancelled, Transitions to FRAME: Schedule View
- ACTION: "Go Back" -> Transitions to FRAME: AppointmentDetail

## FRAME: Reschedule
- REF: design-system/components/time-slot-picker.jsx
- ENTERS_FROM: FRAME: AppointmentDetail
- DATA: Available slots for the same provider.
- ACTION: Select new time -> Updates Appointment.appointment_time, Transitions to FRAME: Schedule View
- ACTION: "Cancel" -> Transitions to FRAME: AppointmentDetail

## FRAME: QuickAdd
- REF: design-system/templates/quick-entry-form.jsx
- REF: design-system/mockups/quick-add-walkin.png
- ENTERS_FROM: FRAME: Schedule View
- DATA: Patient search, reason for visit, provider selection.
- ACTION: "Create Appointment" -> Creates Appointment in Confirmed status, Transitions to FRAME: Schedule View
- ACTION: "Cancel" -> Transitions to FRAME: Schedule View
