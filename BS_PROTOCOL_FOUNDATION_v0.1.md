# BS PROTOCOL — FOUNDATION v0.1

> The operating layer of the foundation. Not ontology (that lives in `BS_SEED_DOCUMENT_v0.3.md`), not technical specification (that lives in `bcs_core_agent_spec.md`). This is the **minimal contour** that any first participant, investor, or grant reviewer must understand identically.

This document is canonical. Russian and French versions exist as `.ru.md` and `.fr.md`. In case of disagreement, this English version prevails.

---

## 1. Operating Definition

**BS Protocol / ARQ MVP** is a minimal collaboration protocol in which participants themselves produce questions, decisions, traces, and rules of interaction, while the system helps route the path from an unclear problem to a recorded result.

> An AI-guided protocol for adaptive collaboration. It helps small groups formulate problems, distribute attention, record decisions, and adapt their rules of working together as complexity grows.

This single paragraph is the canonical external description. It is the language used for Innosuisse, the public landing, and any cold introduction. Anything more elaborate is appendix.

## 2. The Practical Route

A single collaboration cycle is eight steps:

```
Problem → Question → Trace → Signal → Coordination → Action → Result → Rule Update
```

| Step | What happens | Who moves it | Where it lives |
|---|---|---|---|
| **Problem** | An internal unclarity | Any human | Pre-system |
| **Question** | Unclarity becomes a formulated Q | ARQ assists | ARQ bot |
| **Trace** | Q and its first moves are recorded as an artifact | ARQ + author | Persistent storage |
| **Signal** | The Trace becomes a signal to others | System routes | ARQ + cell channel |
| **Coordination** | Several participants pick up, distribute attention | Cell (3–5 ppl) | ARQ + kChat |
| **Action** | Execution of what was decided | Cell | Real world |
| **Result** | A recorded change | Cell | Trace updated |
| **Rule Update** | If a procedural correction was needed, it is entered | Cell via BS-fork | Foundation v.NEXT |

The MVP must demonstrate **at least one full cycle** on a group of 3–5.

## 3. The Three Structural Invariants

These rules are not subject to forking. Without them the protocol collapses.

1. **Question ≠ task.** A Q is a formulated tension, not a request for work. Without this, Q degrades into a Jira ticket and the protocol becomes a task manager.

2. **Spark must carry cost-of-error.** Any proposed direction of resolution must explicitly state "what becomes visible if this hypothesis is wrong." Without this, Trace accumulates opinions instead of verifiable moves.

3. **Initiator ≠ Assembler.** The person who formulated a Q cannot be the one who closes its Trace. This is a structural anti-self-validation invariant. It prevents the cycle from collapsing onto a single person regardless of cell size.

Everything else — windows, formats, roles, deadlines, storage layout — is determined by the cell itself and evolves through the **Rule Update** step (see §2).

## 4. Self-Amending Principle

A cell applies Q/S/T to its own procedure of collaboration. When something breaks or works poorly, that becomes a Q, passes through the cycle, and Rule Update changes the procedure for subsequent cycles.

**What evolves:** collection windows, Trace formats, role composition, routing methods, closure criteria.

**What does not evolve:** the three invariants in §3. They protect the very fact that evolution remains protocol-governed and does not collapse into arbitrariness.

## 5. Where Each Thing Lives

| Layer | Artifact | Purpose |
|---|---|---|
| **Constitutional** | this document + `BS_SEED_DOCUMENT_v0.3.md` | Invariants, philosophy |
| **Runtime** | ARQ Live bot (see `bcs_core_agent_spec.md` and addendum) | Routing algorithm, protocol execution |
| **Surface** | mathr.ch | Explanation of what this is, for whom, how to enter |
| **Cell conventions** | Each cell keeps its own `CELL_LOG.md` | Local rules, evolving via Rule Update |

ARQ Live bot is not "a conversational interface." It is **the conductor from internal unclarity to coordinated action.**

## 6. Vocabulary: Internal ↔ External

The words on which we speak internally do not always work outward. This table is binding.

| Internal | External (Innosuisse, landing, first contact) |
|---|---|
| Стая | Protocol cell / guided collaboration group |
| Нода | Participant / contributor |
| Trace | Recorded decision cycle |
| Spark | Proposed direction |
| Guided elevation of protocol | AI-guided protocol onboarding for adaptive collaboration |
| BS Protocol | Adaptive collaboration protocol (in technical contexts) |
| Forking the rule | Updating cell conventions |
| Свадхарма / vocation | Vocation alignment (in research-audience texts) |

Rule: preserve the live vocabulary internally, translate to architectural and methodological terms externally. Never collapse into "yet another DAO" or "AI tool."

## 7. Phase 0 — What Must Be Proven

Not "that the protocol works at scale." Only three things:

1. **Single-user route.** One person passes through ARQ from fog to a precise Q and a closed Trace.
2. **Small pack flow.** A group of 3–5 completes a full cycle of §2 on a real task.
3. **Live Rule Update.** A cell detects a procedural gap, runs Q/S/T on the procedure itself, updates its `CELL_LOG.md`.

When these three demonstrations exist, we can speak with Innosuisse, first participants, and serious audiences not as utopia but as mechanism.

## 8. Out of Scope for v0.1

- Trace Score mathematics
- Cross-cell routing
- Blockchain anchoring, tokens, anti-Sybil mechanisms
- Governance between cells
- ARQ technical architecture (lives in `bcs_core_agent_spec.md` and its addendum)

These layers open after Phase 0 §7 is closed. Not before.

---

*v0.1 — 2026-05-26. Open for fork under the rules of §2 step 8 + §4.*
