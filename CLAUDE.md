# CLAUDE.md

@docs/narrative.yaml

Behavioral guidelines for implementing projects defined by a `narrative.yaml`. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Read the Narrative First

**The `narrative.yaml` is the spec. Read it before writing any code.**

Before implementing:
- Read `@docs/narrative.yaml` in full to understand the project scope, stack, and architecture.
- Follow `[[references]]` — they form the implementation DAG. Build top-down within each section: `architectural` → `features` → `flourishes`.
- State your assumptions explicitly. If uncertain about intent, ask — don't invent.
- If a simpler approach exists within the constraints of the narrative, say so.

## 2. Respect the DAG

**Implement nodes in dependency order. Never skip ahead.**

- Scaffold `architectural` nodes first: Service → Subsystem → Module.
- Ship `features` only after the `architectural` nodes they reference are in place.
- Layer `flourishes` only after the `features` they enhance are done.
- Ask before starting each top-level node. Implement one node at a time.
- After implementing a node, update its `status` to `review`. Do not advance until confirmed.

## 3. Derive Tests from `value`

**The `value` field is the test spec. No separate test document needed.**

- `architectural` items → unit tests (inputs, outputs, error conditions from `value`).
- `features` items → acceptance/integration tests (given/when/then from `value`).
- `flourishes` items → visual/interaction tests (appearance and behavior from `value`).
- Write `value` descriptions in testable language: "must", "should", "given X returns Y".

## 4. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what the narrative describes.
- No abstractions for single-use code.
- No "flexibility" that wasn't in the narrative.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 5. Surgical Changes

**Touch only what you must. Match the narrative's stated stack.**

- Use only the technologies listed in the `technical` section — treat it as an allowlist.
- Don't "improve" adjacent code or formatting unrelated to the current node.
- If you notice a gap or conflict in the narrative, surface it — don't silently resolve it.
- Every changed line should trace directly to a node in the narrative.

---

**These guidelines are working if:** implementation maps cleanly to the narrative DAG, tests derive directly from `value` fields, and clarifying questions come before implementation rather than after mistakes.
