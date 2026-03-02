---
name: trace-generate
description: >
  Implements a software project from a Trace specification folder. A Trace folder contains
  overview.md, stack.md, personas/ (.persona.md files), domains/ (.domain.md files), and
  flows/ (.flow.md files) that together declare a complete system. This skill reads the Trace,
  produces a phased implementation plan organized by domain dependency, and builds the project
  phase by phase with review gates between each phase.

  Use this skill whenever someone provides a Trace folder, asks to implement/build/generate
  code from a Trace, or has a project folder containing .domain.md, .persona.md, or .flow.md
  files. Also trigger when someone mentions "Trace specification", "Trace folder", or asks to
  "implement from spec" when the spec follows Trace conventions.
---

# Trace Implementation Skill

You are implementing a software project from a Trace specification. Trace is a declarative modeling format where the developer declares *what* a system is and you generate the code. The full specification is available at `references/trace-spec.md` — consult it when you need to look up keyword meanings, formatting rules, or generation guidelines.

The work happens in three phases. Do not skip phases or combine them. Each phase has a clear deliverable and a gate where the user must approve before you continue.

---

## Phase 0: Read and Model

Read every file in the Trace folder in this exact order:

1. `overview.md` — Understand the project's purpose, scope, and global glossary. Note the `design_system` path if one is declared.
2. `stack.md` — Understand the technical choices and SYSTEM DEFAULTS (audit fields, ID strategy, money handling, event delivery, identity model). These defaults apply to every entity unless a domain explicitly overrides them.
3. Every file in `personas/` — Understand who uses the system, what they care about, and critically, their ACCESS rules. Access rules are enforced at the data layer, not just the UI.
4. Every file in `domains/` — Understand entities, attributes, rules, events, constraints, and glossaries. Pay close attention to LISTENS TO declarations — they define the dependency graph between domains.
5. Every file in `flows/` — Understand the screens, frames, actions, and transitions for each persona.

After reading, build a mental model of:

- **Domain dependency graph**: Which domains emit events, and which domains listen? A domain that only emits events has no upstream dependencies. A domain that listens depends on the domain it listens to. This ordering drives the implementation plan.
- **Event chain**: Trace the full event flow from trigger to handler. For example: Appointment status changes → PatientCheckedIn event → Clinical creates a Visit → VisitCompleted event → Billing creates a Claim.
- **Persona-to-domain mapping**: Which personas interact with which domains? This comes from the flow files (each flow declares its persona and domains).
- **STUB rules**: Identify every rule marked STUB. These get a function signature and a placeholder comment only — no implementation.
- **External systems**: Note every EXTERNAL declaration. These become integration points in the plan.

Present a brief summary of this mental model to the user before moving to Phase 1. This confirms you've read everything correctly.

---

## Phase 1: Implementation Plan

Produce a written implementation plan organized as **vertical slices by domain**. Each phase in the plan takes one domain from database schema all the way to UI. The user must approve this plan before any code is written.

### Plan structure

**Foundation phase (always first):**
- Project scaffolding based on stack.md (package.json, framework setup, database connection, ORM config)
- SYSTEM DEFAULTS implementation (audit fields, ID generation, money handling utilities, event outbox table and worker)
- Auth setup based on the identity model declared in stack.md

**Domain phases (ordered by dependency):**
- Domains that only emit events come first (they have no upstream dependencies)
- Domains that listen to events come after the domains they listen to
- Each domain phase includes:
  - Database schema and migrations from ENTITY + MUST HAVE + SYSTEM DEFAULTS
  - Row-level security and permissions from PERSONA + ACCESS
  - Backend validation from RULES (every rule, enforced server-side)
  - Event emission from EVENT + TRIGGER + PAYLOAD
  - Event handlers from LISTENS TO declarations
  - API endpoints
  - Frontend routes from SCREEN sequencing in related flows
  - UI components from FRAME definitions in related flows
  - State management based on flow strategy (persisted wizard state for `normalize`, screen-state transitions for `denormalize`)

**For each domain phase, call out:**
- Which entities are created
- Which rules are enforced and where (backend validation, frontend UX, or both)
- Which events are emitted and which are consumed
- Which STUB rules will get placeholder-only treatment (list them explicitly)
- Which external integrations are involved
- Which personas interact with this domain and what access controls apply

**At the end of the plan, include:**
- A list of all STUB rules across all domains, with a note that these are signature-only
- Open questions — anything in the spec that is ambiguous, contradictory, or missing. Do not guess. Ask.

### Waiting for approval

Present the plan and explicitly ask: "Does this plan look right? Any changes before I start building?" Do not write any implementation code until the user approves. If they request changes, revise the plan and ask again.

---

## Phase 2: Build Phase by Phase

After plan approval, implement one phase at a time. After each phase, present the code and wait for the user to review before continuing to the next phase.

### For each phase:

1. **Implement** the phase according to the plan
2. **Explain the mapping** — for each significant piece of generated code, briefly note which Trace declaration it comes from (e.g., "This validation enforces Appointment.R2: no overlapping appointments for a provider")
3. **Present and wait** — show the code and ask for review. Do not proceed to the next phase until the user says to continue

### Code generation guidelines

**From stack.md:**
- Use the exact technologies declared. Do not substitute frameworks or libraries.
- Apply SYSTEM DEFAULTS to every entity: add id/created_at/updated_at fields, audit with actor_id, store money in cents, etc.

**From domains:**
- Generate database schemas from ENTITY + MUST HAVE declarations
- Enforce every RULE in backend validation. Also reflect rules in the frontend UX (disable buttons when actions aren't valid, show warnings, prevent invalid state transitions)
- Generate event emission logic from EVENT + TRIGGER + PAYLOAD
- Generate event handlers from LISTENS TO declarations. Handlers must be idempotent (per SYSTEM DEFAULTS)
- Respect CONSTRAINTS — if a domain says it doesn't access another domain's data, enforce that boundary in the code architecture

**From personas:**
- Enforce ACCESS rules at the data layer (row-level security, query scoping, API authorization). Access control is never UI-only.
- Use persona descriptions and research findings to inform UX decisions (component choices, copy, flow)

**From flows:**
- Generate routes from SCREEN declarations
- Generate components from FRAME declarations
- Wire up actions to domain operations and screen/frame transitions
- For `normalize` strategy: multi-screen workflow with persisted state across routes
- For `denormalize` strategy: single-screen SPA with frame transitions and no URL changes
- When REF points to design system assets, use them as guidance for component selection and layout patterns

**STUB rules:**
- Generate only a function signature and a placeholder comment like `// STUB: Cancellation fee logic — to be implemented separately`
- Do not implement the logic. Do not guess at the implementation.

---

## Hard Rules

These are non-negotiable. Violating any of these means the implementation is wrong.

1. **Never invent domain concepts.** If an entity, attribute, rule, event, or term is not declared in a .domain.md file, do not create it. The Trace is the source of truth. If you think something is missing, surface it as an open question — do not fill the gap yourself.

2. **Never skip RULES enforcement.** Every RULE in every domain must be enforced in backend validation. Rules that affect user actions must also be reflected in the frontend UX (disabled states, warnings, validation messages). A rule that exists only in backend code but is invisible to the user is incomplete. A rule that exists only in the UI but not the backend is a security hole.

3. **Never ignore ACCESS rules.** Every ACCESS declaration in every persona file must be enforced at the data layer — row-level security, query scoping, API-level authorization. Hiding a button in the UI is not access control. If a persona "can only see their own appointments," that constraint must exist in the database query or RLS policy, not just the frontend.

4. **Never implement STUB rules.** When a rule is marked STUB, generate a function signature with the correct parameters and a placeholder comment. Nothing more. The implementation will be planned and built separately.

5. **Never proceed without plan approval.** Phase 1 ends with a question, not with code. Wait for explicit approval before writing implementation code. If the user has concerns, address them first.

---

## Reference

The complete Trace specification is at `references/trace-spec.md`. Consult it for:
- Full keyword glossary (Section 4)
- Syntax and formatting rules (Section 3)
- The "Normal UI" logic for normalize vs. denormalize strategies (Section 5)
- AI generation rules (Section 7)
- Cross-domain communication patterns (Section 2)
