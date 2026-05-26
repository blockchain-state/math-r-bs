# Math.R / BS Protocol

Canonical conceptual and operational documents for the **BS (Blockchain State)** protocol and its initiatives. This repository is the single source of truth for the foundation and specifications. Implementation code lives in sibling repositories (see *Related repositories* below).

> Languages: the canonical version of any document is the English file (`*.md`). Russian (`*.ru.md`), French (`*.fr.md`) and forthcoming German (`*.de.md`) are companion versions. In case of disagreement, the English file prevails.

## Document layers

### Constitutional layer — invariants and ontology

- `BS_PROTOCOL_FOUNDATION_v0.1.md` — operational foundation: definition, route, three structural invariants, self-amending principle. **Read this first.** Versions: `.ru.md`, `.fr.md` (DE pending).
- `BS_SEED_DOCUMENT_v0.3.md` — root ontological document (RU).
- `BS_ACTIVATION_GUIDE_v0.1.md` — activation guide (RU).

### Specification layer — runtime

- `bcs_core_agent_spec.md` — original technical specification of the ARQ Live agent (RU, dated 2026-02-27).
- `ARQ_SPEC_AUDIT_v0.1.md` — audit of the above against `BS_PROTOCOL_FOUNDATION_v0.1.md`. Identifies conflicts and the minimal patch path.
- `ARQ_SPEC_ADDENDUM_v0.2.md` — patch document re-defining core primitives to align with foundation. Authoritative override for §2.4, §5.1, §3.2 of the original spec. Read together with the audit.

### Auxiliary

- `mvlp3_schematic-1.html`, `schematic_header.html` — MVLP3 schematics.
- `LinkedIn_Final_TrustGame_v3.md` + `assets/TrustGame_LinkedIn_Header.png` — TrustGame materials.

## Reading order for newcomers

1. `BS_PROTOCOL_FOUNDATION_v0.1.md` — what we are doing, in one page.
2. `BS_SEED_DOCUMENT_v0.3.md` — why this matters, the civilizational frame.
3. `bcs_core_agent_spec.md` + `ARQ_SPEC_AUDIT_v0.1.md` — how the runtime is being built and where the current spec needs updating.

## Related repositories

- [arq](https://github.com/blockchain-state/arq) — ARQ Live Bot + War Room implementation (private). Code only. Spec lives in this repository.
- [site](https://github.com/blockchain-state/site) — public hub `mathr.ch`. Renders the surface explanation of the foundation.
- [mars-protocol](https://github.com/blockchain-state/mars-protocol) — MARS Protocol track.
- [aether-vortex](https://github.com/blockchain-state/aether-vortex) — VAYU / aether-vortex engineering track.
