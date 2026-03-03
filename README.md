# Trace

Trace is a declarative modeling specification for AI-driven development. You declare *what* a system is (its infrastructure, its people, its domain logic, and its user experience) in a structured folder of human-readable text files. The AI generates the implementation.

Trace is designed for what developers most need to learn and focus on in the age of AI: how systems are architected, what problems they are solving, and who they are solving them for.

## How It Works

A Trace is a folder. It contains five layers that fully describe a software system:

```
your-project/
├── overview.md          # What this system is and who it's for
├── stack.md             # Technical choices and system defaults
├── personas/            # The people who use the system
│   └── *.persona.md
├── domains/             # Data models, rules, events, constraints
│   └── *.domain.md
└── flows/               # Screens, interactions, and user experience
    └── *.flow.md
```

| Layer | What It Defines |
|---|---|
| **Overview** | Project purpose, global glossary, design system reference |
| **Stack** | Language, frameworks, database, system defaults (audit fields, ID strategy, money handling) |
| **Personas** | Who uses the system, what they care about, what data they can access |
| **Domains** | Entities, attributes, business rules, events, cross-domain constraints |
| **Flows** | Screen-by-screen UX for each persona, with actions, data, and transitions |

The standard is not highly proscriptive. It instead relies on the "L" in LLM, allowing architects and engineers to largely define the spec in natural language.

It's recommended to use a trace to generate a plan or plans. For large projects with many domains and flows, you can ask your agent to split the plan in vertical slices or, since flows mark which domains they depend on, focus on implementation of certain flows first.

If you make changes after generation, it's recommended to ask the LLM to update the trace to match the code, keeping the trace as a "source of truth".

## Get Started

The fastest way to understand Trace is to look at a real example:

**[`examples/fish-family-medicine/`](examples/fish-family-medicine/)** — A complete Trace for a doctor's office with scheduling, clinical visits, and billing. Three personas, three domains, two flows.

Feed the folder to an LLM and ask it to implement the system.

## Learn More

- **[`SPEC.md`](SPEC.md)** - The full Trace specification: design philosophy, project structure, keywords, formatting rules, and AI generation rules.
- **[Normal UI](https://dontimitate.dev/normalui)** - Read the full book on Normal UI for free.

## Agent Skills

The **[`skills/`](skills/)** folder contains Agent Skills for working with Trace projects. These are reusable prompts that can be loaded into AI coding assistants (like Claude Code) to automate common workflows.

- **`trace`** — Helps write a Trace spec (with educational notes for newer devs). Generates a software project from a Trace folder. Reads the spec, produces a phased implementation plan organized by domain dependency, and builds phase by phase with review gates.

It's recommended to use the Agent Skill when planning and implementing a trace. The skill will evaluate your trace and present any open questions the trace does not cover prior to implementation.

## License

Apache 2.0
