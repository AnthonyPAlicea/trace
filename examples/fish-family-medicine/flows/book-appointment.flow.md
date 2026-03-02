---
flow: BookAppointment
strategy: normalize
persona: Patient
domains:
  - Scheduling
---

# WORKFLOW: Book a New Appointment
> GOAL: Low cognitive load. Guide the patient through booking without medical jargon.

## SCREEN: Choose Reason
- REF: design-system/templates/card-selection-grid.jsx
- REF: design-system/examples/friendly-option-picker.png

This is the patient's first decision point. Keep it warm and simple. Patients don't think in medical categories, so use plain language like "I'm not feeling well" instead of "Acute Symptom Visit." Icons help here.

- DATA: A simplified list of reasons (e.g., "Routine Checkup", "I'm Feeling Sick", "Follow-up Visit", "Something Else").
- These map to internal reason_for_visit values but are displayed in plain language.
- ACTION: Select "Something Else" -> Goes to SCREEN: Describe Concern
- ACTION: Select any standard reason -> Goes to SCREEN: Choose Provider

## SCREEN: Describe Concern
- REF: design-system/templates/single-field-form.jsx
- DATA: Free-text field for patient to describe their concern.
- ACTION: "Next" -> Goes to SCREEN: Choose Provider
- ACTION: "Back" -> Goes to SCREEN: Choose Reason

## SCREEN: Choose Provider
- REF: design-system/templates/list-selection.jsx
- REF: design-system/mockups/provider-card-layout.png
- DATA: Provider.name, Provider.specialty, Provider.accepting_new_patients
- Only show providers relevant to the selected reason.
- If the patient has a primary provider, pre-select them.
- ACTION: Select a provider -> Goes to SCREEN: Choose Time
- ACTION: "I don't have a preference" -> Auto-selects next available provider, Goes to SCREEN: Choose Time
- ACTION: "Back" -> Goes to SCREEN: Choose Reason

## SCREEN: Choose Time
- REF: design-system/templates/time-slot-picker.jsx
- REF: design-system/mockups/time-slot-grid.png
- DATA: Available appointment slots based on Provider.availability and duration rules.
- New patient appointments show only 30+ minute slots (Appointment.R3).
- ACTION: Select a time slot -> Goes to SCREEN: Confirm
- ACTION: "None of these work" -> Goes to SCREEN: Waitlist Offer
- ACTION: "Back" -> Goes to SCREEN: Choose Provider

## SCREEN: Waitlist Offer
- REF: design-system/templates/info-action-page.jsx
- DATA: Explanation that no slots are available.
- ACTION: "Join Waitlist" -> Creates waitlist entry, Goes to SCREEN: Waitlist Confirmation
- ACTION: "Try a Different Provider" -> Goes to SCREEN: Choose Provider
- ACTION: "Cancel" -> Goes to SCREEN: Exit

## SCREEN: Waitlist Confirmation
- REF: design-system/templates/confirmation-message.jsx
- Confirmation that patient has been added to waitlist.
- ACTION: "Done" -> Goes to SCREEN: Exit

## SCREEN: Confirm
- REF: design-system/templates/review-summary.jsx
- REF: design-system/mockups/booking-summary.png
- DATA: Summary of reason, provider, date/time.
- ACTION: "Book Appointment" -> Creates Appointment in Requested status, Goes to SCREEN: Success
- ACTION: "Change Something" -> Goes to SCREEN: Choose Reason
- ACTION: "Cancel" -> Goes to SCREEN: Exit

## SCREEN: Success
- REF: design-system/templates/success-message.jsx
- Confirmation message with appointment details.
- ACTION: "Add to Calendar" -> Generates calendar event.
- ACTION: "Book Another" -> Goes to SCREEN: Choose Reason
- ACTION: "Done" -> Goes to SCREEN: Exit
