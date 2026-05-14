# Narrative (yaml)

A simple standard to remember features and architectural patterns for agents & humans at the intersection of design, product, and engineering specialties.

This document is:
 - like a Product Review Document (PRD)
 - like User Personas, User Stories, and User Journey Maps
 - like a Software Design Document
 - more than just a Software Architecture Haiku

 But is:
 - not an Architecture Decision Record (ADR)
 - not a Market Requirements Document (MRD) 
 - not Epic and Sprint Planning
 - not a Software Technical Specification
 - not API documentation

We are documenting the vibes across specialties such that an LLM can write all the code -- this is an interdisciplinary project/product narrative.  It basically **IS** the software in English where the micro-decisions are left to the implementation loop.


Additional Motivation:
 - Microsoft Research is showing 25% corruption by foundation models [LLMs Corrupt Your Documents When You Delegate](https://arxiv.org/abs/2604.15597)

## Getting Started

See [template.narrative.yaml](template.narrative.yaml) for a full reference see the example file.

### Using with Claude Code

Copy the [`CLAUDE.md`](CLAUDE.md) from this repo into your project root. It instructs Claude to read `@docs/narrative.yaml` automatically at the start of every session and follow the implementation DAG, derive tests from `value` fields, and stay within your declared tech stack.

```bash
curl -sL https://raw.githubusercontent.com/otcom-dev/narrative.yaml/main/CLAUDE.md -o CLAUDE.md
```

To bootstrap your project with the example file:

```bash
mkdir -p docs && curl -sL https://raw.githubusercontent.com/otcom-dev/narrative.yaml/main/template.narrative.yaml -o docs/narrative.yaml
```

Or browse the `examples/` directory for full worked examples:
- [todo-webapp.narrative.yaml](examples/todo-webapp.narrative.yaml) — a React/Next.js todo web app
- [todo-tui.narrative.yaml](examples/todo-tui.narrative.yaml) — the same app reimagined as a Go TUI

You can use this prompt to have your LLM / AI assistant update it for your project / product:
```
Read ./docs/narrative.yaml. Then ask me questions — one at a time — to help me
customize it for the product I am building. Start with the general section to
understand the problem and audience, then work through technical, architectural,
features, and flourishes. Replace all placeholder values with what you learn.
Do not make assumptions — ask when unsure.
```


Once your narrative is complete, use this prompt to begin implementation:
```
Read ./docs/narrative.yaml. Resolve the implementation DAG:
1. Identify all [[references]] in the architectural section and implement top-down —
   scaffold each Service, then its Subsystems, then its Modules, then its Components, ...
2. Once architectural nodes are in place, implement features top-down — ship each
   Capability as a thin slice first, then deepen with its Features and Sub-features.
   For each feature, only reference architectural components already implemented.
3. Once features are implemented, layer in flourishes top-down within each concern.
4. After implementing each node, write tests derived from its `value` description.
   Architectural nodes → unit tests (inputs, outputs, error conditions).
   Feature nodes → acceptance/integration tests (given/when/then from user behavior).
   Flourish nodes → visual and interaction tests.
Ask me before starting each top-level node. Implement one node at a time, updating
its status to 'review'.  Do not proceed to the next node until the current one is
implemented and tested.
```

### Structure

A `narrative.yaml` file is designed to document the intersection of design, product, and engineering specialties.  It is This provides guidance for humans and  organized into top-level sections:

| Section | Question Answered | Description |
|---|---|---|
| `definitions` | What is what? | Vocabulary/glossary of terms used throughout the file |
| `general` | What is in scope? | Project / Product identity and purpose metadata |
| `technical` | Build with which legos? | Technology stack interpreted as allowlist / defaults |
| `architectural` | How it is built? | Herarichal decomposition of the project / product engineering design |
| `features` | How it works? | Herarichal decomposition of feature with statuses |
| `flourishes` | How it feels? | Design language, visual engagement, rendering styles, etc... |

### Cross-referencing with `[[...]]`

Any named item in any section can be referenced elsewhere in the file using double-bracket syntax:

```yaml
value: >
  The [[Backend]] writes data to the [[Storage]] component.
  This is a [[Feature]] of the [[App]].
```

References resolve to the `name` field of any entry in any section. They serve as a lightweight hyperlink between concepts — keeping descriptions concise while preserving meaning. An LLM reading the file should follow references to build full context before acting.

Typically you define your vocabulary in `definitions` first (we have provided a start), then scope the project in `general`.  Then reference both in the herarichal decomposition in `architectural`.  Finally, freely reference `[[names]]` across `features` and `flourishes`.

### Implementation DAG

The `[[...]]` references form a **Directed Acyclic Graph (DAG)** of implementation dependencies. Each node is a named item; each reference is a directed edge meaning "depends on" or "is built on top of."

Each of `architectural`, `features`, and `flourishes` is itself a **tree** — items within a section can reference other items in the same section, creating a hierarchy of increasing specificity:

```
definitions / general / technical
        ↓
   architectural                     ← tree: Services → Subsystems → Modules
        ↓
     features                        ← tree: Pillar/Theme → Features → Sub-features
        ↓
    flourishes                       ← tree: Design system → Patterns → Details
```

Cross-section references (e.g. a `feature` referencing an `architectural` node) create the edges that connect the three trees into a single DAG.

All three sections are decomposed top-down and built top-down — but the sections themselves must be sequenced:

- **`architectural`** — scaffold the Service first, fill in Subsystems, then Modules. Defines the foundation everything else depends on.
- **`features`** — ship the Capability first, deepen with Features, then Sub-features. Gated by the `architectural` nodes they reference.
- **`flourishes`** — a forest of independent trees (one per design concern), each built top-down. Every flourish is gated by the feature it enhances — no feature, no flourish.

The only bottom-up constraint is **between** sections: `architectural` must be in place before the `features` that reference it, and `features` must be done before the `flourishes` that enhance them.

Items whose `[[references]]` are all marked `done` can be implemented in parallel. A `status` field on any item marks its node as complete, allowing the graph to be pruned dynamically as work progresses.

### Deriving Tests

The `value` field of every item is also its implicit test specification — no separate test document is needed. Each layer of the DAG maps to a different layer of testing:

- **`architectural` items** → unit tests. The `value` describes the contract of a Module or Subsystem: its inputs, outputs, and error conditions. An LLM should derive unit tests directly from this description.
- **`features` items** → acceptance and integration tests. The `value` describes observable user behavior. An LLM should derive "given/when/then" acceptance tests from Feature and Sub-feature descriptions.
- **`flourishes` items** → visual and interaction tests. The `value` describes how an enhancement should look or behave. An LLM should derive visual regression or interaction tests from these descriptions.

Write `value` descriptions in testable language — "must", "should", "given X returns Y", "when Z happens then W is visible" — so that both humans and LLMs can read the narrative and know exactly what to verify.


## Linting

You can validate your `narrative.yaml` file with [yamllint](https://yamllint.readthedocs.io/). Because narrative files use long prose `value` fields, disable the line-length rule:

```bash
yamllint -d '{extends: default, rules: {line-length: disable}}' docs/narrative.yaml
```

Or add a `.yamllint.yaml` to your project root:

```yaml
extends: default
rules:
  line-length: disable
```

Then run:

```bash
yamllint docs/narrative.yaml
```


## Projects Adhering to Narrative (yaml)


The following projects follow or are using the `narrative.yaml` standard:

| Project | Description |
|---|---|


## Contributing

Narrative (yaml) is an open standard. Contributions are welcome:

1. **Spec improvements** — Open an issue or PR to refine the standard
4. **Adopt Narrative (yaml)** — Build your App following the standard and submit a PR to be listed
