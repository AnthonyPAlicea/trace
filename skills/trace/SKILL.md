---
name: trace
description: Work with Trace, a declarative modeling specification for AI-driven development. Use this skill whenever someone wants to design a new software system, create Trace specification files (overview.md, stack.md, personas, domains, flows), learn system architecture, domain modeling, user flows, or personas, build or generate code from an existing Trace folder, or mentions Trace, .domain.md, .flow.md, .persona.md, or declarative modeling. This skill handles two modes: authoring new Traces (for both experienced developers and those learning system design) and implementing from existing ones.
---
Trace is a declarative modeling specification for AI-driven development. A Trace is a folder of structured markdown files that declare what a system is: its infrastructure, personas, domain logic, and user experience. The developer shapes the system. The AI generates the code.

Before doing anything, read the full specification:

```references/trace-spec.md```

Then determine which mode to operate in.

## Mode Selection
**Author mode:** The user wants to design a new system or doesn't have a Trace yet. They might say "I want to build an app for...", "help me design...", "I have an idea for...", "create a Trace for...", or wants to learn domain modeling, system architecture, user flows, or personas.

Load ```references/trace-write.md``` to help the user write Trace files.

**Implement mode:** The user already has a Trace folder and wants to generate code from it. They might say "build this", "implement this Trace", "generate code from...", or provide a folder of .domain.md, .flow.md, and .persona.md files.

Load ```references/trace-generate.md``` to help the user plan or execute implementation from their Trace.

If it's unclear, ask: "Are we designing a new system or building from an existing Trace?"

It's common for a session to start in author mode and transition to implement mode. When the user is satisfied with their Trace and says something like "ok build it" or "let's implement this", switch to implement mode naturally.
