# ARQ Core Agent Spec — Addendum v0.2

> Patch document. Aligns `bcs_core_agent_spec.md` (2026-02-27) with `BS_PROTOCOL_FOUNDATION_v0.1.md` (2026-05-26). Based on findings in `ARQ_SPEC_AUDIT_v0.1.md`.
>
> **Rule of precedence.** Where this addendum and the original spec disagree, this addendum prevails. Where this addendum is silent, the original spec remains in force.

---

## 1. Authoritative source of core primitives

The original spec uses `Quest`, `XP`, `quest_history`, `quest_submit` as core primitives. From v0.2 onward, the authoritative core primitives are defined in `BS_PROTOCOL_FOUNDATION_v0.1.md` §3:

- **Question (Q)** — replaces `Quest` everywhere in the operational layer.
- **Spark (S)** — new primitive. Did not exist in original spec.
- **Trace (T)** — replaces the role of `ActionRecord` *as a closed cycle artifact*. `ActionRecord` continues to exist as a lower-level audit primitive (one row per event), but is no longer the unit of collaboration.

The three structural invariants of the foundation are enforced by ARQ as code-level checks, not as social conventions.

## 2. Replacements in the original spec

### 2.1 §2.4 Quest Module → Question / Spark / Trace Module

The original Quest Module is replaced by a single module with three sub-flows. Module-level responsibilities:

| Sub-flow | Replaces in original | New responsibility |
|---|---|---|
| `question_intake` | Quest selection (`/quest`) | Receives raw user input, AI-assists to formulate a Q meeting the format rules (§3 below) |
| `spark_collection` | Quest submission (`/submit`) | Collects Sparks from other participants. Each Spark must carry `cost_of_error`. No anonymous Sparks |
| `trace_assembly` | Verification (auto/peer/committee) | One Assembler (≠ Initiator) synthesizes Sparks into a sealed Trace |

The `/quest` command is renamed `/q`. The `/submit` command is removed (Sparks are published, not submitted; Traces are sealed, not submitted).

### 2.2 §5.1 Quest YAML format → Trace markdown format

The directory `data/quests/` is removed. Predefined top-down "quest catalogs" contradict invariant 1 (Q ≠ task) — there is no operator-side library of work; every Q is user-originated.

In its place, the runtime maintains a directory tree per cell:

```
data/cells/<cell-id>/traces/
  ├── open/      Q-NNNN-<slug>.md     ← awaiting Sparks
  ├── assembling/ Q-NNNN-<slug>.md    ← Sparks collected, Assembler at work
  ├── sealed/    T-NNNN-<slug>.md     ← closed cycle
  └── stale/     Q-NNNN-<slug>.md     ← timed out without Sparks
data/cells/<cell-id>/CELL_LOG.md      ← local conventions, evolved via Rule Update
```

Numbering is monotonic per cell, common to Q and T (a Q keeps its number when sealed as a T).

### 2.3 §2.5 Reputation Module — display deferred

The reputation computation in §2.5 is **kept as code** (decay-based math is correct), but the bot **does not display Trace Score in v0.2**. Until 100 sealed Traces have accumulated cell-wide, the bot displays only:

- `traces_initiated`: count of Q the participant has authored
- `sparks_published`: count of Sparks the participant has published
- `traces_assembled`: count of Traces the participant has assembled
- `sparks_validated / sparks_falsified`: explicit, separately tracked

This avoids gamification optics that contradict the foundation's framing ("not a points game").

### 2.4 §3.2 Onboarding flow — first action is a Q, not a quiz

The original 5-step onboarding (multiple-choice quiz + domain selection + micro-task) is restructured. New onboarding:

1. **Welcome + 3-sentence framing** (kept from original §3.2)
2. **Display name** (kept)
3. **"What unclarity is alive for you right now?"** — open input. ARQ helps formulate the user's first Q.
4. **First Q is sealed as `Q-0001` for this participant.** It does not get Sparks immediately. It is parked in `traces/open/` as a signal others may pick up.
5. **Navigation:** bot shows open Q from other participants. User can publish a Spark on any.

The onboarding produces a real Trace cycle entry from minute one, not a synthetic "completed onboarding" event.

### 2.5 §2.4 Domains — seed list, not closed taxonomy

The original 6 domains (Development / Coordination / Thinking / Education / Communication / Arbitration) are kept as **seed** for cold start, not as a fixed taxonomy.

A cell may introduce a new domain through Rule Update (foundation §4). New domains require:
- One Q-level case that does not fit existing domains
- One sealed Trace demonstrating the new domain in practice
- Entry in `CELL_LOG.md`

This satisfies the foundation's principle that domains are emergent, not predefined.

## 3. New format specifications

### 3.1 Question (Q) format

A valid Q is a markdown file with this front matter:

```yaml
---
id: Q-NNNN
slug: <kebab-case-summary>
cell: <cell-id>
initiator: <handle>
opened: <ISO-8601>
state: open | assembling | sealed | stale
domain: <seed-domain or cell-defined>
---
```

Followed by markdown body:

- **Title** — 5 to 15 words, in the form of a question.
- **Tension** — 1 to 3 paragraphs (max 300 words). What is the gap between what is and what should be.
- **Context** — what is already known, what has been tried.
- **Stake** — what changes if Q is resolved, what is blocked if not.

ARQ's `question_intake` flow refuses inputs that fail this format and proposes reformulation.

### 3.2 Spark (S) format

A Spark is appended to its parent Q's markdown body, under section `## Sparks`:

```markdown
### Spark by <handle> at <ISO-8601>

**Hypothesis.** <1 paragraph, 50–200 words>

**Basis.** <what supports this — experience, reference, experiment, analogy>

**Cost of error.** <what becomes visible if this hypothesis is wrong>
```

ARQ's `spark_collection` flow rejects Sparks where the `Cost of error` field is empty or generic ("we'll see", "depends"). Specific falsifiable predictions only.

### 3.3 Trace (T) format

When a Trace is sealed, it gains an additional section:

```markdown
## Assembly by <handle> at <ISO-8601>

<which Sparks were taken into the resolution, which were set aside and why,
the final resolution OR a reference to a successor Q if reformulated>

## Outcome

<what changed in the world: artifact, decision, new protocol,
or explicit "Q reformulated into Q-NNNN+1">

## Closure type

resolution | reformulation | dissolution
```

The bot then moves the file from `traces/assembling/` to `traces/sealed/` and assigns a `T-NNNN` identifier (same NNNN as the originating Q).

## 4. Code-level enforcement of invariants

The runtime enforces all three invariants in code:

| Invariant | Enforcement |
|---|---|
| Q ≠ task | `question_intake` LLM check + structural format check (title is interrogative, body has Tension/Context/Stake) |
| Spark must carry cost-of-error | `spark_collection` rejects empty/generic `cost_of_error` field |
| Initiator ≠ Assembler | `trace_assembly` refuses to seal if `assembler_handle == initiator_handle` for that Q |

These checks are tests in `tests/test_invariants.py` and must remain green for any deployment.

## 5. What in the original spec is unchanged

The following sections of `bcs_core_agent_spec.md` continue to apply as written:

- §1.1 Overall architecture (Router / Knowledge / Profile / Reputation / Analytics)
- §1.2 Architecture principles
- §2.1 Router Module — *with intent renames: `quest_*` → `q_*`*
- §2.2 Knowledge Module — including system prompt (excellent tonal alignment with foundation §6)
- §2.3 Profile Module — *with field renames: `quest_history` → `participation_log`, `actions` retained as low-level audit*
- §2.5 Reputation math — *display deferred, computation kept*
- §2.6 Analytics Module — *event types updated to Q/S/T events*
- §3.1 First contact flow — unchanged
- §3.4 Question about BCS flow — unchanged
- §4 Technical stack — unchanged
- §6+ all subsequent sections — unchanged

## 6. Migration path for implementation

1. Implement the runtime on top of the original spec.
2. Treat this addendum as the authoritative override for §2.4, §5.1, §3.2, and the parts noted in §5 above.
3. After Phase 0 §7 of the foundation is proven (one full cycle on a 3–5 person cell), produce `bcs_core_agent_spec_v0.3.md` as a consolidated rewrite, retiring both this addendum and the 2026-02-27 original.

Do not attempt a consolidated rewrite before Phase 0 is closed. The system is still learning what its own primitives should be.

## 7. Open questions deferred to later addenda

- Cross-cell Q routing (when Q in one cell becomes Signal to another).
- Cell formation and membership management (currently expected to happen out-of-band via kChat invites; ARQ only serves Q/S/T inside an existing cell).
- Conflict between Sparker and Initiator over Assembly result (currently routed to Reformulation closure type; may need explicit appeal mechanism later).
- Publication pipeline from sealed Trace to public Encyclopedia entry on `mathr.ch/traces/`.

---

*v0.2 — 2026-05-26. Open for fork under foundation §2 step 8.*
