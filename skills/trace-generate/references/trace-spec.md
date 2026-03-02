# Trace: A Declarative Modeling Specification for AI-Driven Development

**Trace** is the practice of declaring *what* a system is: its infrastructure, its people, its domain logic, and its user experience, in a structured folder of human-readable text files that serve as a specification precise enough for an AI to generate a complete implementation from, or for a human development team to build from directly. The developer shapes the system. The AI generates the code.

**Tagline:** Shape the System, Generate the Code.

Trace is designed for what developers most need to learn and focus on in the age of AI: how systems are architected, what problems they are solving, and who they are solving them for.

---

## 0. Design Philosophy: Designed for LLMs

Trace is built for Large Language Models, not parsers.

Traditional DSLs need rigid syntax. A parser can only understand exactly what it's been programmed to recognize. An LLM is the opposite. It understands natural language natively. It can infer meaning from context (though not always perfectly). It can read "States move forward only" and know what that means without a formal state machine grammar.

This means:

- **Use natural language wherever possible.** Write rules, constraints, and descriptions in plain English. The LLM will understand them. Do not invent formal syntax for things English already expresses clearly.
- **Use structure only for organization, not meaning.** Headers, keywords, and frontmatter exist to help the LLM (and humans) *find* information quickly. The meaning itself lives in the natural language beneath them.
- **Trust the LLM to interpret.** "Cancellation fee applies if status is InProgress" is a perfectly valid rule. You do not need `IF status == InProgress THEN apply(CancellationFee)`. The moment the language starts looking like code, it has failed.
- **The convention is lightweight on purpose.** Just enough scaffolding to be scannable and organized. No more.

When in doubt, write it the way you'd explain it to a smart colleague. If they'd understand it, the LLM will too.

---

## 1. Project Structure

A **Trace** is a folder. It contains an overview, a stack definition, and three subfolders for personas, domains, and flows.

```
fish-family-medicine/
├── overview.md
├── stack.md
├── personas/
│   ├── patient.persona.md
│   ├── front-desk.persona.md
│   └── provider.persona.md
├── domains/
│   ├── scheduling.domain.md
│   ├── clinical.domain.md
│   └── billing.domain.md
└── flows/
    ├── book-appointment.flow.md
    ├── front-desk-dashboard.flow.md
    └── patient-check-in.flow.md
```

### `overview.md` (The Big Picture)

- **Purpose:** A plain language description of what this system is, who it's for, and what problem it solves. Also contains the **global GLOSSARY**: terms that mean the same thing across all domains.
- **Scope:** One per project. Written for humans first, but gives the AI essential context about intent and scope.
- **AI Behavior:** Read this first. Use it to understand the overall purpose so that all generation decisions serve the project's goals. Use the global glossary as the default meaning for terms unless a domain glossary overrides it.

### `stack.md` (Infrastructure)

- **Purpose:** Defines the global technical choices (Language, Database, Frameworks) and **SYSTEM DEFAULTS**: scaffolding conventions that apply to all entities (audit fields, ID strategy, money handling, event delivery, identity model).
- **Scope:** One per project.
- **AI Behavior:** Use this to determine *how* to implement the logic. Apply SYSTEM DEFAULTS to every generated entity unless a domain explicitly overrides them.

### `personas/` (People)

- **Purpose:** Each file defines a persona: who they are, what they care about, and what data they can access. Persona files can also contain user research findings, interview notes, jobs-to-be-done, and other details that inform design decisions.
- **Scope:** Declared once, referenced by both Domain and Flow files.
- **AI Behavior:** Use these to determine permissions, row-level access, and which screens belong to which users. Richer research details help the AI make better UX decisions.

### `domains/` (Data & Logic)

- **Purpose:** Each file defines a single domain: its entities, attributes, invariants, events, and constraints. A domain is self-contained. Its terms, entities, and rules don't leak into other domains. Each domain can include a **GLOSSARY** that defines terms as they exist in that specific domain, overriding the global glossary when meanings differ.
- **Scope:** Pure logic. UI-agnostic. References Personas for data access rules.
- **AI Behavior:** Use these to generate Database Schemas, TypeScript Interfaces, and Backend Validation Logic. When a term appears in both the domain glossary and the global glossary, the domain glossary wins within that context.

### External References: Design System

A Trace folder can reference an external **design system** that lives outside the project. This could be a component library, a set of page templates, image mockups, code examples, or any combination. Flows reference design system assets to guide the AI (or a human developer) toward the intended look and feel.

The design system is not prescriptive. It provides examples and references, not rigid templates. The AI should use these references as guidance for component selection, layout patterns, and visual consistency, not as exact specifications to reproduce.

The design system location is declared in `overview.md` frontmatter:

```
---
name: Fish Family Medicine
design_system: ../design-system/
---
```

Flow files then reference specific assets using the `REF` keyword on any SCREEN or FRAME.

### `flows/` (User Experience)

- **Purpose:** Each file defines a user experience: screens, frames, and interactions for a specific persona and workflow.
- **Scope:** References one or more Domain files and one or more Persona files. REF keywords point to assets in the design system declared in `overview.md`.
- **AI Behavior:** Use these to generate Frontend Routes, State Management stores, and UI Components. When a design system reference is provided, use it as guidance for component selection and layout patterns.

---

## 2. Constraints & Boundaries

Trace supports multiple `.domain.md` files in a single project. Each domain file is self-contained: its terms, entities, and rules are internally consistent and don't leak into other domains.

When domains need to interact, the rules are simple: declare the relationship in natural language. The LLM will respect it.

### Cross-Domain Communication

A domain file can declare that it listens to events from another domain, and what it does in response. Use the `LISTENS TO:` keyword followed by plain English describing the behavior.

### Context Boundaries

Use the `CONSTRAINTS:` keyword at the top of a domain file to declare, in natural language, what this domain can and cannot do. These are hard rules the AI must never violate.

Constraints can express:

- **Data isolation:** "This domain does not directly access Scheduling data."
- **Communication rules:** "Billing only learns about visits through the VisitCompleted event."
- **Term definitions:** "'Patient' in this context means an account with insurance info, not a clinical record."
- **Behavioral boundaries:** "This domain never auto-submits claims. All claims require review."

You do not need formal syntax for these. Write them the way you'd explain them to a new team member. The LLM will interpret and enforce them.

### Example: Two Related Domains

> This example illustrates a critical domain modeling insight: an "Appointment" and a "Visit" are NOT the same thing. An Appointment is a scheduled slot. It can be requested, confirmed, rescheduled, or cancelled. A Visit is what happens when a patient actually shows up. It has clinical notes, a diagnosis, and generates a billing claim. A naive developer collapses these into one "Appointment" table and spends months untangling the mess. Good domain modeling catches this distinction before any code is written.

**File:** `domains/scheduling.domain.md`

```
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
```

**File:** `domains/clinical.domain.md`

```
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
```

**File:** `domains/billing.domain.md`

```
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
```

---



## 3. Syntax & Formatting Rules

To maintain readability and parsability, you must adhere to the "Clean Caps" standard.

- **Headers define Scope:**
  - `#` (H1) defines the File Subject.
  - `##` (H2) defines the Major Object (Entity, Screen).
  - `###` (H3) defines the Sub-Object (Frame).

- **Capitalized Keys define Properties:**
  - Do NOT use bolding (`**`) for keywords.
  - Use All-Caps followed by a colon (e.g., `MUST HAVE:`, `RULES:`).

- **Indentation defines Ownership:**
  - Lists under a keyword belong to that keyword.

- **System Notes:**
  - Use blockquotes (`>`) starting with `SYSTEM NOTE:` to denote technical implementation details that override business logic.

---

## 4. Keywords Glossary

### Overview Keywords (`overview.md`)

| Keyword | Meaning |
|---|---|
| `GLOSSARY` | Global definitions of terms that mean the same thing across all domains. Domain-level glossaries override these when meanings differ |

### Stack Keywords (`stack.md`)

| Keyword | Meaning |
|---|---|
| `SYSTEM DEFAULTS` | Scaffolding conventions applied to all entities: audit fields, ID strategy, money handling, event delivery, identity model. Domains can override specific defaults |

### Persona Keywords (`.persona.md`)

| Keyword | Meaning |
|---|---|
| `PERSONA` | A role or user type in the system |
| `DESCRIPTION` | Natural language description of who this person is, what they care about, and what they're trying to accomplish |
| `ACCESS` | Natural language rules for what data this persona can see and modify |

### Domain Keywords (`.domain.md`)

| Keyword | Meaning |
|---|---|
| `GLOSSARY` | Definitions of terms as they exist in this specific domain. Overrides the global glossary in `overview.md` when meanings differ across contexts |
| `ENTITY` | A database table or core model |
| `MUST HAVE` | List of fields/columns and their types |
| `RULES` | Business invariants. Logic that must always be true |
| `CONSTRAINTS` | Natural language boundaries for this domain: what it can and cannot do, how it communicates, and how its terms differ from other domains |
| `RELATIONSHIPS` | Connections to other entities (Has Many, Belongs To) |
| `EVENT` | A domain event emitted by the system |
| `TRIGGER` | The condition that causes an event |
| `PAYLOAD` | The data carried by an event |
| `LISTENS TO` | Declares that this domain reacts to an event from another domain or an external system, followed by natural language describing the response |
| `EXTERNAL` | Declares an external system dependency for this domain (e.g., `EXTERNAL: Twilio SMS, Stripe`). Makes integrations visible in the implementation plan |
| `STUB` | Placed after a rule label (e.g., `R5 STUB:`) to indicate the AI should generate a function signature and placeholder but not implement the logic. The implementation will be planned and built separately |

### Flow Keywords (`.flow.md`)

| Keyword | Meaning |
|---|---|
| `STRATEGY` | Either `normalize` (step-by-step) or `denormalize` (all-in-one) |
| `PERSONA` | References a Persona defined in a `.persona.md` file. Declares who this screen is for |
| `SCREEN` | A distinct URL route or full-page view |
| `FRAME` | A state of a Screen. A Screen can transition between Frames without changing the URL |
| `DATA` | Fields referenced from one or more Domain files |
| `ACTION` | A user interaction and its consequence. Every action declares what happens (data change, validation, etc.) AND where to go next (transition to a Screen or Frame). Actions are the edges of the flowchart |
| `REF` | A reference to a design system asset: component, template, mockup, image, or code file. Resolves against the design_system path in overview.md. Not prescriptive, but gives the AI (and human devs) a concrete example of what this screen or frame should look or behave like |
| `VALIDATION` | Checks against RULES defined in the Domain file |

---

## 5. The "Normal UI" Logic

You must apply specific architectural patterns based on the `strategy` defined in the `.flow.md` frontmatter.

### Pattern A: normalize

- **Definition:** Splitting a complex workflow into a sequence of focused screens. Each screen can branch to different screens based on user actions.
- **Structure:** A flowchart of `SCREEN`s. Each `ACTION` declares which `SCREEN` it transitions to. Screens can branch, merge, loop back, or dead-end.
- **Output:** A multi-screen workflow with state preserved across routes. Not necessarily linear. The path through screens depends on user choices and data conditions.

### Pattern B: denormalize

- **Definition:** Consolidating related tasks into a single screen that transitions between multiple states (Frames). Each frame can branch to different frames based on user actions.
- **Structure:** A `SCREEN` containing a flowchart of `FRAME`s. Each `ACTION` declares which `FRAME` it transitions to.
- **Output:** A Dashboard or Single Page Application (SPA). The screen transitions between Frames without changing the URL. Each Frame represents a different state of the screen, with different data visible, different actions available. Frames can branch, loop back, or dead-end.

---

## 6. Examples

### Example 1: The Stack File

**File:** `stack.md`

```
---
stack: Fish-Family-Medicine-V1
---

BACKEND:
- Runtime: Node.js (v20)
- Framework: Fastify
- Language: TypeScript

DATA:
- Database: Supabase (Postgres)
- ORM: Prisma
- Realtime: Supabase Channels

FRONTEND:
- Framework: TanStack Start
- State: Zustand
- Styling: Tailwind CSS

SYSTEM DEFAULTS:
- All entities have id (UUID), created_at, and updated_at unless stated otherwise.
- All writes are audited with actor_id.
- Money is stored in cents as integers. Displayed as USD.
- Auth users are managed by Supabase Auth. Patient.user_id, Provider.user_id, and FrontDesk.user_id link to auth.users.
- Domain events are persisted in an outbox table and processed by a worker. Handlers must be idempotent.
```

### Example 2: Persona Files

**File:** `personas/patient.persona.md`

```
---
persona: Patient
---

# PERSONA: Patient

DESCRIPTION:
- A person receiving care at the practice.
- Wide age range. Many are not tech-savvy.
- Cares about getting an appointment quickly and not being surprised by bills.
- Often anxious when visiting the doctor. Clarity reduces anxiety.

ACCESS:
- Can only see their own appointments and visit history.
- Can see their provider's name and specialty. Cannot see other patients' data.
- Can cancel or reschedule their own appointments.
- Cannot see clinical notes until the provider releases them.

## Research Findings

### From user interviews, Dec 2024
- 4 of 6 patients said they've accidentally booked the wrong appointment type because the options were confusing.
- Patients over 60 strongly preferred calling over using an app but were open to "something simple."
- Multiple patients mentioned frustration with not knowing if their insurance was verified before arriving.

### Jobs to Be Done
- When I need to see my doctor, I want to book the right type of appointment without having to know medical terminology.
- When I arrive at the office, I want check-in to be fast so I'm not standing at the desk feeling awkward.
- After my visit, I want to understand what I owe before I get a surprise bill.
```

**File:** `personas/front-desk.persona.md`

```
---
persona: FrontDesk
---

# PERSONA: Front Desk Staff

DESCRIPTION:
- Manages the daily flow of the office. Juggles phone, walk-ins, and the schedule simultaneously.
- Not clinical. Does not need to see diagnosis or notes.
- Cares about keeping the schedule full, avoiding no-shows, and moving patients through quickly.
- Power user. Uses the system all day, every day.

ACCESS:
- Can see all appointments across all providers.
- Can create, reschedule, and cancel appointments.
- Can check patients in.
- Cannot see clinical notes or diagnosis codes.
- Can see insurance verification status but cannot modify insurance details.
```

**File:** `personas/provider.persona.md`

```
---
persona: Provider
---

# PERSONA: Provider (Doctor / NP / PA)

DESCRIPTION:
- The clinician seeing patients.
- Extremely time-constrained. Every extra click costs patient time.
- Cares about seeing the right information before walking into the room.
- Writes clinical notes and assigns diagnosis codes.

ACCESS:
- Can see appointments and visits for their own patients.
- Can see and edit clinical notes only for their own visits.
- Cannot modify the schedule directly. Requests changes through front desk.
- Can see patient insurance status (to inform treatment decisions) but cannot modify billing.
```

### Example 3: The Domain File

> The domain files for this example are in Section 2 (Constraints & Boundaries), which shows all three domains: Scheduling, Clinical, and Billing.

### Example 4: The Flow File (Denormalized)

**File:** `flows/front-desk-dashboard.flow.md`

```
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
- ENTERS_FROM: FRAME: AppointmentDetail
- DATA: Available slots for the same provider.
- ACTION: Select new time -> Updates Appointment.appointment_time, Transitions to FRAME: Schedule View
- ACTION: "Cancel" -> Transitions to FRAME: AppointmentDetail

## FRAME: QuickAdd
- REF: design-system/templates/quick-entry-form.jsx
- ENTERS_FROM: FRAME: Schedule View
- DATA: Patient search, reason for visit, provider selection.
- ACTION: "Create Appointment" -> Creates Appointment in Confirmed status, Transitions to FRAME: Schedule View
- ACTION: "Cancel" -> Transitions to FRAME: Schedule View
```

### Example 5: The Flow File (Normalized)

**File:** `flows/book-appointment.flow.md`

```
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
- DATA: Free-text field for patient to describe their concern.
- ACTION: "Next" -> Goes to SCREEN: Choose Provider
- ACTION: "Back" -> Goes to SCREEN: Choose Reason

## SCREEN: Choose Provider
- DATA: Provider.name, Provider.specialty, Provider.accepting_new_patients
- Only show providers relevant to the selected reason.
- If the patient has a primary provider, pre-select them.
- ACTION: Select a provider -> Goes to SCREEN: Choose Time
- ACTION: "I don't have a preference" -> Auto-selects next available provider, Goes to SCREEN: Choose Time
- ACTION: "Back" -> Goes to SCREEN: Choose Reason

## SCREEN: Choose Time
- DATA: Available appointment slots based on Provider.availability and duration rules.
- New patient appointments show only 30+ minute slots (Appointment.R3).
- ACTION: Select a time slot -> Goes to SCREEN: Confirm
- ACTION: "None of these work" -> Goes to SCREEN: Waitlist Offer
- ACTION: "Back" -> Goes to SCREEN: Choose Provider

## SCREEN: Waitlist Offer
- DATA: Explanation that no slots are available.
- ACTION: "Join Waitlist" -> Creates waitlist entry, Goes to SCREEN: Waitlist Confirmation
- ACTION: "Try a Different Provider" -> Goes to SCREEN: Choose Provider
- ACTION: "Cancel" -> Goes to SCREEN: Exit

## SCREEN: Waitlist Confirmation
- Confirmation that patient has been added to waitlist.
- ACTION: "Done" -> Goes to SCREEN: Exit

## SCREEN: Confirm
- DATA: Summary of reason, provider, date/time.
- ACTION: "Book Appointment" -> Creates Appointment in Requested status, Goes to SCREEN: Success
- ACTION: "Change Something" -> Goes to SCREEN: Choose Reason
- ACTION: "Cancel" -> Goes to SCREEN: Exit

## SCREEN: Success
- Confirmation message with appointment details.
- ACTION: "Add to Calendar" -> Generates calendar event.
- ACTION: "Book Another" -> Goes to SCREEN: Choose Reason
- ACTION: "Done" -> Goes to SCREEN: Exit
```

---

## 7. AI Generation Rules

When given a Trace folder, you should:

1. **Read `overview.md` first** to understand the project's purpose and scope.
2. **Read `stack.md` second** to determine implementation technology.
3. **Read all files in `personas/` third** to understand who uses the system and their data access rules.
4. **Read all files in `domains/` fourth** to understand the data model, invariants, events, and constraints.
5. **Read all files in `flows/` fifth** to understand how each persona interacts with the domain.
5. **Generate code that respects all four layers:**
   - Database schemas and migrations from ENTITY + MUST HAVE
   - Row-level security and permissions from PERSONA + ACCESS
   - Validation logic from RULES
   - Event handlers from EVENT + TRIGGER + PAYLOAD
   - Routes and navigation from SCREEN sequencing
   - Component architecture from FRAME definitions
   - State management from strategy (persisted wizard state for `normalize`, screen-state transitions for `denormalize`)
6. **Never invent domain concepts.** If it's not in the `.domain.md`, don't create it.
7. **Never violate invariants.** Every RULE must be enforced in both backend validation and frontend UX (e.g., disable buttons, show warnings).
8. **Never ignore access rules.** Every PERSONA's ACCESS declarations must be enforced at the data layer, not just hidden in the UI.
9. **Ask clarifying questions** when the model is ambiguous or contradictory rather than guessing.
