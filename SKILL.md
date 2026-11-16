---
name: spd
description: "SPD (Systematic Problem Decomposer): Categorizes, decomposes, and solves complex tasks using Cynefin triage, MECE issue trees, Theory of Constraints bottleneck isolation, Hierarchical Task Analysis (HTA), and First Principles error recovery. Invoke with /spd [depth=1|2|3] or automatically on complex debugging, architecture, and root-cause analysis."
argument-hint: "[depth=1|2|3] <problem or task>"
---

# SPD — Systematic Problem Decomposer

Use this protocol to systematically break down ambiguous, multi-step, or high-risk tasks before executing mutations.

## 1. Argument Parsing & Tree Depth Configuration

Inspect the user prompt or `/spd` invocation for an explicit depth override (e.g., `depth=1`, `depth=3`, `--depth 2`, `shallow`, `deep`). If none is specified, use **Standard (`depth=2`)**.

| Depth Setting | Alias | Tree Structure & Scope | Best For |
| :--- | :--- | :--- | :--- |
| **`depth=1`** | `shallow` | 1 level (2–4 top-level buckets). Immediate TOC selection and flat 3-step HTA plan. | Smaller/faster models, tight context windows, or localized bugs. |
| **`depth=2`** *(Default)* | `standard` | 2 levels (2–4 domains $\rightarrow$ 2–3 specific leaf hypotheses/components per active branch). Full HTA plan with verification gates. | General engineering, multi-file debugging, and system configuration. |
| **`depth=3`** | `deep` | 3 levels (Domain $\rightarrow$ Subsystem $\rightarrow$ Atomic Mechanism/Edge Case). Cross-branch dependency check + nested HTA with rollback rules. | High-reasoning models, distributed systems, or major architectural design. |

*Hard Guardrail:* Never exceed `depth=4` or more than `4` child branches per node. Expand deeper levels only along plausible branches—prune ruled-out branches early to conserve tokens.

---

## 2. Execution Protocol

Follow these five phases in strict order. Do not skip directly to code edits or system mutations unless Phase 1 classifies the task as **CLEAR**.

### Phase 1: Cynefin Triage (Classify Before Acting)
Evaluate cause-and-effect predictability before calling mutation tools:
- **CLEAR:** Deterministic, well-understood pattern with an obvious fix. $\rightarrow$ *Bypass Phases 2–4 and execute directly.*
- **COMPLICATED:** Deterministic system where cause and effect require structured analysis. $\rightarrow$ *Proceed to Phase 2 using the configured `depth`.*
- **COMPLEX:** Unfamiliar environment, race conditions, emergent behavior, or opaque external state. $\rightarrow$ *Forbid mutations. Run 1–3 read-only diagnostic probes first to gather ground truth, then re-classify as COMPLICATED.*
- **CHAOTIC:** Active failure, outage, or data corruption requiring immediate containment. $\rightarrow$ *Stabilize/rollback first, then re-classify.*

### Phase 2: MECE Decomposition (Map the Problem Space)
Build an issue/component tree up to the active `depth`:
- **Mutually Exclusive (ME):** Zero overlap between sibling branches.
- **Collectively Exhaustive (CE):** Together, branches cover 100% of the problem space (e.g., *Inputs/Parsing*, *Core Logic*, *State/Storage*, *External Interfaces/Environment*).
- For **diagnostic tasks**, leaf nodes must be falsifiable root-cause hypotheses.
- For **build/design tasks**, leaf nodes must be decoupled architectural deliverables.

### Phase 3: Theory of Constraints (Isolate the Bottleneck)
Evaluate the leaf nodes from Phase 2 and select **one**:
- Identify the single node that acts as the primary bottleneck or highest-probability root cause.
- Explicitly mark all other branches as `DEFERRED`.
- **Rule:** Do not refactor, optimize, or touch deferred branches while resolving the active constraint.

### Phase 4: Hierarchical Task Analysis (HTA Execution Plan)
Translate the isolated bottleneck into an explicit control structure before acting:
- **Goal:** Target state required to resolve the active constraint.
- **Sub-goals:** Numbered atomic steps (`1`, `2`, `3`...).
- **Plan Logic:** Execution order (`1 -> 2 -> 3`), conditional fallbacks (`if step 2 fails -> check X`), and a mandatory **Verification Gate** (an exact command, test, query, or assertion) that must pass before marking the sub-goal complete.

### Phase 5: Checkpoint & Iterate
Once the Verification Gate passes:
1. Summarize the verified state change in 2–3 lines (drop raw diagnostic noise from focus).
2. Re-evaluate the MECE tree: if the top-level objective is met, conclude; otherwise, run Phase 3 again to isolate the next bottleneck.

---

## 3. Circuit Breaker: First Principles Reset

**Trigger:** A sub-goal fails **twice** (2 consecutive failed edits, tool errors, or failed Verification Gates).

1. **HALT:** Stop patching the current approach immediately.
2. **EXPOSE ASSUMPTIONS:** List every implicit assumption being made about configs, environment variables, library behavior, permissions, or data formats.
3. **VERIFY GROUND TRUTH:** Run raw read-only checks (inspect raw logs, actual loaded configs, process state, environment, or minimal inputs) to prove or falsify each assumption.
4. **REBUILD:** Discard invalidated assumptions and construct a new Phase 4 HTA plan from verified facts only.

---

## 4. Compact Output Format

Use this concise scaffold before executing tools so reasoning remains auditable and token-efficient:

```text
[SPD | Depth: <N> | Cynefin: <DOMAIN>]
- MECE Tree:
  - <Branch 1>
    - <Leaf 1.1>
- Active Constraint (TOC): <Selected Leaf> (Deferred: <Other Branches>)
- HTA Plan:
  1. <Sub-goal 1> -> Verify: <Exact check>
  2. <Sub-goal 2> -> Verify: <Exact check>
```
