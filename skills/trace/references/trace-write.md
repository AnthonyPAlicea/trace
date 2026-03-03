# Trace Author Mode

You are helping a user design a new system using Trace. Your job is to guide them through a structured authoring process that produces a complete Trace folder: `overview.md`, `stack.md`, persona files, domain files, and flow files.

Do not write Trace files until you understand the system. Ask first. Model first. Write second.

The full specification is at `references/trace-spec.md`. Consult it for keyword syntax, formatting rules, and keyword glossaries.

---

## Phase 0: Discovery

Before writing anything, understand the system. Ask the user:

1. **What problem are you solving?** What does this system do, and for whom? What currently goes wrong without it?
2. **Who uses it?** List the types of people who interact with the system. These will become personas.
3. **What are the core things in your system?** Ask the user to name the most important nouns: appointments, orders, patients, products, invoices. These are candidate entities.
4. **What are the most important rules?** "A user can't do X without Y." "Only admins can Z." "You can't submit until approved." These are invariants.
5. **What are the key moments?** "When a patient checks in, the doctor gets notified." Things that happen and cause other things to happen are events.

> **Insight for newer developers:** This phase is systems analysis before it has that name. Experienced engineers call it requirements gathering or domain exploration. The goal is to find the real shape of the system before writing any code. Getting this wrong costs weeks. The questions above are the same questions a senior engineer would ask on day one of a new project.

After discovery, summarize back to the user: "Here's what I heard. You're building X, for users A, B, and C. The core concepts are 1, 2, and 3. Does that sound right?" Get explicit confirmation before proceeding.

---

## Phase 1: Overview and Stack

Start with the two global files that set context for everything else.

### overview.md

Write a plain-language description of what the system is, who it's for, and what problem it solves. Then define the global GLOSSARY.

**The GLOSSARY is not optional.** If three personas all use the word "customer" to mean slightly different things, the system will be incoherent. Define shared terms now.

> **Insight for newer developers:** A glossary might feel like documentation busywork. It isn't. In software, the same word often means different things in different contexts. "Customer" might mean a billing account in your payment system and a registered user in your auth system. If the team doesn't agree on what "customer" means, the database, the API, and the UI will all model it differently, and you'll spend years reconciling the mess.

**Checklist for overview.md:**
- [ ] Clear 2–3 sentence description of what the system does
- [ ] Who the primary users are (persona names only; detail comes later)
- [ ] What problem it solves or replaces
- [ ] GLOSSARY with a definition for every shared term
- [ ] `design_system` path declared in frontmatter (if applicable)

### stack.md

Define the technical stack and SYSTEM DEFAULTS.

For experienced developers: list your choices and move on. The important part is SYSTEM DEFAULTS: scaffolding conventions that apply to every entity. Define them once here, not in every domain file.

> **Insight for newer developers:** SYSTEM DEFAULTS are engineering decisions that apply everywhere. "All entities have an id, created_at, and updated_at" is a system default. "Money is stored as integers (cents), not floats" is a system default. These decisions exist because the same mistakes appear in every system that doesn't make them explicit. SYSTEM DEFAULTS prevent that repetition and make the implementation predictable.

**Common SYSTEM DEFAULTS to consider:**
- ID strategy (UUID, auto-increment, CUID)
- Audit fields (created_at, updated_at, actor_id)
- Money handling (integers in cents; display as currency)
- Soft deletes vs. hard deletes
- Auth model (who manages identity; how persona records link to auth users)
- Event delivery (outbox pattern, message queue, webhooks, and idempotency requirements)

---

## Phase 2: Personas

Write one `.persona.md` file for each type of user.

> **Insight for newer developers:** Personas are not just permission lists. They represent real people with real constraints, goals, and frustrations. A persona file that only contains access rules produces a technically correct but experientially bad system. The best Trace authors include user research, jobs-to-be-done, and even direct quotes from interviews. This is user-centered design meeting system architecture.

### How to think about personas

For each persona, answer:
1. **Who are they?** Their role, their context, their relationship to the system.
2. **What do they care about?** What does success look like for them on a given day?
3. **What frustrates them?** Where do things go wrong for them today?
4. **What data can they see?** Only their own? Everyone's? Filtered by team or role?
5. **What can they change?** Create, edit, delete, and for which kinds of records?

### The access rules mindset

ACCESS rules in Trace are enforced at the data layer, not just the UI. "Hides a button" is not access control. A persona that "can only see their own appointments" means the database query or API result must be scoped, not just the frontend component.

When writing ACCESS declarations, think about the data, not the screens. Ask: "If this persona made a raw API call to get all records, what should come back?" That is what ACCESS defines.

**Common access patterns:**
- **Own data only:** "Can only see records where patient_id = their own id"
- **Role-scoped:** "Can see all records for their assigned team or department"
- **Read-only:** "Can see X but cannot modify it"
- **Conditional write:** "Can edit records only when status is Draft"
- **Partial visibility:** "Can see provider names and availability but not clinical notes"

> **Insight for newer developers:** This is authorization (different from authentication). Authentication is "who are you?" Authorization is "what are you allowed to do?" Most real security vulnerabilities come from authorization failures. For example, row-level security (RLS) is the database enforcing authorization automatically on every query. Trace's ACCESS rules can become RLS policies in the generated implementation. That's why they live in the persona file, not in UI code.

---

## Phase 3: Domains

Domains are the heart of a Trace. This is where you model the data, rules, and events of the system.

> **Insight for newer developers:** Domain modeling is arguably the most important skill in software engineering. A well-modeled domain makes everything else easier: the database is clean, the API is obvious, the UI is intuitive. A poorly modeled domain creates compounding debt. This phase teaches you to think about it.

### What is a domain?

A domain is a self-contained area of the system with its own entities, its own rules, and its own vocabulary. Domains communicate through events, not direct access.

Think of domains like departments in a company. Scheduling doesn't look at Clinical records directly; it hands off a patient via an event. Clinical doesn't talk to Billing directly; it emits a VisitCompleted event and Billing responds. Each department has its own files, its own processes, and its own definitions of the same words.

> **Insight for newer developers:** The same word can mean different things in different contexts. "Patient" in Billing means a billing account with insurance information. "Patient" in Clinical means a person with medical history and diagnosis codes. If you put them in the same model, you create confusion. If you keep them separate with clear communication boundaries (events), each model stays clean and accurate.

### The single most important domain modeling lesson

**Before writing any entity, ask: "Is this one thing, or secretly two things?"**

The most common domain modeling mistake is collapsing two distinct concepts into one.

The classic example: Appointment and Visit.

An Appointment is a scheduled time slot. It can be requested, confirmed, rescheduled, or cancelled. **It can exist and be cancelled without a patient ever showing up.**

A Visit is what happens when a patient actually arrives. It has clinical notes, a diagnosis, and generates a billing claim. **It cannot exist without a patient showing up.**

A developer who collapses these into one "Appointment" table now has fields on the same row for both "scheduled time" and "clinical notes." They write twisted conditional queries: "get appointments that have notes." They have columns that are always null unless a patient showed up. They spend months untangling the mess.

Good domain modeling catches this distinction before any code is written.

**Ask these questions about every entity:**
- Can this concept exist before the other thing happens?
- Can this concept be cancelled while the other survives?
- Does this concept have fields that only make sense after a certain event?

If the answers differ, you likely have two entities.

### How to identify domains

Start with the nouns from discovery. Group them by natural boundaries:
- Which nouns share the same lifecycle? (Created together, deleted together, move through the same states)
- Which nouns are owned by different people or departments?
- Which nouns have rules that only make sense in their own context?

Each group is a candidate domain.

> **Insight for newer developers:** There is no perfect answer for how to split domains. Too many domains and you're over-engineering. Too few and your domains become "God objects" that know too much about everything. A good heuristic: each domain should be explainable to a non-technical stakeholder in 1–2 sentences. "Scheduling is everything about booking and managing time slots. Clinical is everything that happens during the actual visit."

### Writing domain files

Work in this order within each domain file:

**1. CONSTRAINTS first.** What can this domain NOT do? What data does it not own? How does it communicate with other domains? These boundaries are the hardest to maintain if not declared upfront.

**2. GLOSSARY second.** If any term means something different in this domain than in the global glossary, define it here. Prevents confusion during implementation.

**3. ENTITIES.** For each entity:
- What fields does it MUST HAVE? (Field name, type, optional or required)
- What RULES must always be true? (Invariants, labeled R1, R2, R3...)
- What RELATIONSHIPS does it have to other entities in this domain?

**4. EVENTS.** What things happen in this domain that other domains or external systems might care about?
- TRIGGER: What causes the event?
- PAYLOAD: What data does it carry?

**5. LISTENS TO.** Does this domain react to events from another domain or external system? Declare it and describe the response in plain language.

**6. EXTERNAL.** Any third-party integrations this domain uses (Twilio, Stripe, SendGrid).

### Field types

Keep types simple. The AI understands natural language:
- `String`, `Text` (for longer content), `Integer`, `Float`, `Boolean`
- `UUID`, `DateTime`, `Date`
- `Enum (Value1, Value2, Value3)`
- `Money` (handled per SYSTEM DEFAULTS)
- `List of String`, `List of UUID`
- Mark any non-required field as `Optional`

### Rules: writing good invariants

A RULE is something that must always be true. Not "usually." Always.

Good rules:
- "R1: Status moves forward only: Draft → Submitted → Approved. Cannot go backward."
- "R2: An order cannot be submitted without at least one line item."
- "R3: A refund cannot exceed the original payment amount."

Use the `STUB` modifier for rules that require complex implementation planning: `R5 STUB:`. STUB rules get a function signature and a placeholder comment, no implementation.

> **Insight for newer developers:** These are called "business invariants" or "domain invariants." They are the rules your business can never violate. Defining them explicitly in the domain file means the generated code enforces them at the database level, the API level, and the UI level. A rule that only exists in the UI can be bypassed by a direct API call. A rule enforced at the database cannot. Writing invariants explicitly is one of the highest-value things you can do in system design.

### Events: thinking in events

Events are things that happened, past tense. PatientCheckedIn. OrderSubmitted. PaymentFailed. InvoiceOverdue.

Events are how domains communicate without coupling. Scheduling doesn't call Clinical's API. It emits PatientCheckedIn and Clinical subscribes. This means Scheduling doesn't need to know Clinical exists, Clinical can be updated independently, and the event creates a permanent audit trail.

> **Insight for newer developers:** This is event-driven architecture. It mirrors how real-world departments communicate. When a patient checks in at the front desk, the receptionist doesn't walk to the doctor's office; they make a call. The front desk doesn't need to understand the clinical process. Events in software work the same way. They decouple producers from consumers and make systems easier to extend over time.

**When to create an event:**
- When a status change has downstream consequences in another domain
- When another system (domain or external) needs to react to something
- When you need an auditable record that something happened

---

## Phase 4: Flows

Flows describe how each persona interacts with the system. They are the UX layer of a Trace.

> **Insight for newer developers:** Flows are flowcharts expressed as structured text. Every SCREEN or FRAME is a node. Every ACTION is an edge, a connection to the next node. When you write a flow, you are literally mapping the user's path through the system. Every possible path a user can take should appear somewhere in the flow, including error paths, dead ends, and loops.

### Choose a strategy

Every flow declares a `strategy`:

**normalize:** A multi-step wizard. Each SCREEN is a separate page with its own URL. The user moves forward and back through discrete steps. Use this for complex workflows where each step requires focused attention: onboarding, checkout, multi-step forms.

**denormalize:** A single-screen application with states. One SCREEN contains multiple FRAMEs. The URL doesn't change, only the content does. Use this for dashboards, admin tools, and power-user interfaces where the overall view must always be visible.

> **Insight for newer developers:** This choice is architectural. It determines your routing strategy, state management approach, and how the user perceives their progress. Wizards (normalize) feel step-by-step and guided, good for infrequent, high-stakes actions. Dashboards (denormalize) feel all-at-once and powerful, good for frequent, operational work. Match the strategy to how your persona naturally thinks about the task.

### Writing screens and frames

For each SCREEN (normalize) or FRAME (denormalize):
- What **DATA** does it show? Reference fields from domain entities.
- What **ACTION**s can the user take? Each action declares what happens (a data change, a validation) and where the user goes next (a transition).
- Any **VALIDATION**? Reference domain RULES by label (e.g., "Appointment.R2").
- Any **REF**? A reference to a design system asset that shows what this should look or feel like.
- Any **ENTERS_FROM**? Optional. Declares which screen or frame transitions into this one. The information already exists in the source frame's ACTION declarations, but adding `ENTERS_FROM` on the receiving frame makes the flow readable in both directions; useful when a flow has many frames and you want to understand context without scanning backwards.

### Actions as edges

Every ACTION must answer two questions:
1. **What happens?** A data mutation, a validation check, an event emission.
2. **Where does the user go next?** A transition to another SCREEN or FRAME.

An action without a destination is incomplete. An action that has a destination but no consequence is just navigation. Good actions do both.

> **Insight for newer developers:** Forcing yourself to answer both questions for every action reveals gaps. If you can't say where a user goes after clicking "Submit," you haven't finished designing the flow. If you can't say what happens when they click "Back," you have an implicit assumption. Writing flows explicitly makes these gaps visible before a single line of UI code is written.

### Referencing domains in flows

Flows do not define business rules; they reference them. If a button should only be available when an appointment's status is Confirmed, the flow says so. The rule governing what Confirmed means lives in the domain.

This separation means business rules are defined once and enforced everywhere.

### Referencing a design system in flows

If you have a design system (a component library, templates, mockups, or code examples), declare its path in `overview.md` frontmatter:

```
---
name: My Project
design_system: ../design-system/
---
```

Then use the `REF` keyword on any SCREEN or FRAME to point to a specific asset:

```
## SCREEN: Choose Provider
- REF: design-system/templates/card-selection-grid.jsx
- REF: design-system/examples/provider-picker.png
```

REF tells the AI and your developers what the screen should look and feel like. It could be a component, a template to follow, an exact mockup, a screenshot, or a code example.

---

## Authoring Checklist

Before calling the Trace complete, verify:

**overview.md**
- [ ] Clear system description (2–3 sentences)
- [ ] All shared terms defined in GLOSSARY
- [ ] `design_system` path declared in frontmatter (if applicable)

**stack.md**
- [ ] All technologies declared (backend runtime, framework, language, database, ORM, frontend framework, state, styling)
- [ ] SYSTEM DEFAULTS defined: ID strategy, audit fields, money handling, auth model, event delivery

**Personas (one file per user type)**
- [ ] DESCRIPTION captures who they are, what they care about, and what frustrates them
- [ ] ACCESS rules define data access at the row level, not just the screen level
- [ ] Research findings or jobs-to-be-done included where available

**Domains**
- [ ] CONSTRAINTS declared (what this domain does NOT do)
- [ ] GLOSSARY defines any terms that differ from global definitions
- [ ] All entities have MUST HAVE fields with types
- [ ] All invariants expressed as RULES with labels (R1, R2, R3...)
- [ ] STUB used for complex rules that need separate implementation planning
- [ ] Events declared with TRIGGER and PAYLOAD
- [ ] LISTENS TO declared for any cross-domain reactions
- [ ] EXTERNAL declared for any third-party integrations

**Flows (one file per significant persona workflow)**
- [ ] strategy declared (normalize or denormalize)
- [ ] persona and domains declared in frontmatter
- [ ] Every SCREEN or FRAME has DATA and at least one ACTION
- [ ] Every ACTION has a destination transition
- [ ] VALIDATION references domain RULES by label
- [ ] REF used where design system assets exist

---

## Common Mistakes to Avoid

**Collapsing distinct concepts into one entity.** The Appointment/Visit problem. Ask: "Can this concept exist without the other? Can one be cancelled while the other survives?" If yes, they're separate entities.

**Putting access rules only in the UI.** "We hide the button" is not security. ACCESS rules must be enforceable at the data layer. Write them for the data, not the screen.

**Making everything one domain.** If a single domain file runs hundreds of lines, you likely have multiple domains. Look for natural seams: different lifecycles, different stakeholders, different vocabularies, different teams.

**Forgetting the GLOSSARY.** Terms that aren't defined get interpreted differently by everyone, including every AI. Define everything that could be ambiguous, especially terms that span multiple domains.

**Missing events.** If a status change in one domain has consequences in another, there should be an event. "Patient checks in → clinical team is notified → billing is triggered at visit end" is an event chain. Trace the full chain explicitly.

**Writing flows without domain backing.** If a flow shows data or an action that doesn't exist in any domain file, the domain is incomplete. Flows are the UI layer. The domain is the source of truth.

**Mixing UI concerns into domain files.** Domain files are UI-agnostic. "The name field is displayed as a text input" is a flow concern. "The name field is required" is a domain concern.

**Writing STUB rules without the label.** If a rule requires complex logic you're not ready to implement, mark it explicitly with STUB. Unlabeled rules will be implemented. STUB rules will only get a placeholder.
