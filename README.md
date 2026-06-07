# Project Prestidigitonium

> A domain-agnostic auditor pattern for role-based AI governance systems.

Project Prestidigitonium is a plug-and-play specification for adding an **auditor** alongside any role in a role-based AI governance system — without inheriting the role's blind spots, and without losing the domain fluency that makes audits substantive rather than merely procedural.

The pattern is designed to drop into existing role-based systems by way of a **separate auditor role file** that declares the audited role as a loaded input. No new infrastructure. The auditor knows the audited role's domain because it reads that role's file at boot; the auditor's own rules, posture, and authority live in its own file so the auditor's reasoning isn't bound to the same text as the role it audits.

## What's in this repo

| File | Purpose |
|---|---|
| `auditor-pattern-spec.md` | The canonical specification. Encompassment principle + four mechanisms (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation) + naming + authority + scope limits + integration requirements. |
| `template-auditor-role-file.md` | Copy-pasteable skeleton for the separate auditor role file, with annotated guidance on frontmatter, naming, key-check categories, per-criterion clauses, and cross-rule obligations. |
| `audit-corpus-spec.md` | Structure, growth rules, and access discipline for the asymmetric corpus — the body of materials the auditor loads but the audited role does not. Includes active-learning hooks (forward-compatible v0.1.1 + v0.2.0 location update). |
| `adoption-guide.md` | Seven-step walkthrough from "role file with rules" to "conformant auditor role file alongside it." Includes a Migration from v0.1.x section for adopters of the same-file pattern. |
| `conformance-criteria.md` | Nineteen criteria (seventeen Required, two Recommended) for verifying that an implementation realizes the pattern. Includes verdict structure and review procedure. |
| `CHANGELOG.md` | Version history. |
| `LICENSE` | Apache 2.0. |

## The problem this solves

A role file specifies how a builder should reason about its domain. If a defect lives inside that specification — an unstated assumption, a missing trigger, a conflated category — a reader who treats the specification as authoritative cannot see the defect. The defect is invisible *because* it is the lens through which the reader perceives. Adding an "auditor lens" that reads the same specification with a different intent does not solve this: same words cannot catch what the words do not say.

Naive auditor designs sit at one of two failure modes:

- **Same-file lens** — auditor and builder read the same rules. Domain fluency preserved; correlated blind spots inherited.
- **Fully-independent reviewer** — auditor knows nothing about the role. Blind spots avoided; domain fluency lost.

Project Prestidigitonium resolves the tension via **separate-file architecture with declared inclusion**: the auditor lives in its own role file; the audited role file is declared as a loaded input. Surface familiarity (auditor reads the audited role's file); underlying independence (auditor's own rules and stance live elsewhere).

## v0.2.0 architectural revision

v0.1.x of this specification committed to a *same-file* architecture (audit lens as `# §Audit-Variant` section in the audited role's file). v0.2.0 **inverts** that commitment to **separate files**. The four mechanisms survive the inversion; their operationalization moves from "section in the audited role file" to "structure of the separate auditor file."

Adopters of v0.1.x can stay there — the `archive/v0.1.x-same-file-architecture` branch preserves it verbatim. v0.2.0 is the canonical direction going forward; the `main` branch tracks it. Migration guidance is in `adoption-guide.md` §"Migration from v0.1.x".

The reasoning for the inversion: same-file preserved domain fluency at the cost of shared source text, which is the structural cause of correlated blind spots. Mechanism-level mitigations addressed the symptom; separate-file addresses the cause.

## Key terms (glossary)

- **Builder** — the role being audited. The thing that *does the work*. (In Aire shape: named `<project>-<role>` like `sketch-operator` or `aire-smith`.)
- **Auditor** — the separate auditor role file. Named `<audited-role-slug>-auditor.role.md`; display name `<Project> <Role> Auditor`. The thing that *checks the work*.
- **Audited role file** — the role file the auditor audits. Loaded by the auditor as an input via the `audits:` frontmatter field.
- **Auditor role file** — the auditor's own role file. Has frontmatter `audits:` and `follows_pattern:` declaring the pattern adoption.
- **Reciprocal frontmatter pointers** — the audited role file carries `audited_by:`; the auditor file carries `audits:`. Both pointers required for v0.2.0 conformance.
- **Encompassment** — the structural property that the auditor's knowledge is a proper superset of the builder's. In v0.2.0, made explicit by declared input loading.
- **Asymmetric corpus** — materials the auditor loads that the builder does not. Prior findings, failure post-mortems, drift catalogs, the auditor role file's own body, project-specific asymmetric materials.
- **Adversarial default** — the auditor's epistemic stance. Default verdict: refuted; flips to confirmed only on evidence surviving refutation.
- **Independent re-derivation** — derive-then-compare on key checks. Reading the audited file's claim before deriving is not permitted on these checks.
- **Active-learning hooks** — frontmatter fields and structural conventions that make the corpus programmatically analyzable by future tooling. Documented in v0.1.1 of the corpus spec; extensions deferred until accumulated real corpora provide evidence.
- **Key-check categories** — the role-specific list of checks that require derive-then-compare (e.g., section presence, frontmatter completeness, governance reference resolution, re-measurement, re-classification).

## Why a separate repository

This specification is governance-pattern material, not project-specific code. It is published as a standalone repository so that:

- It can evolve independently of any one project's release cadence.
- Projects can pin to a specific spec version and upgrade deliberately.
- Adopters across organizations can reference the same canonical pattern.
- It can be cited in role-file provenance as a stable artifact.

This follows the precedent of [Aire-TOC](https://github.com/LachrymaGhost/Aire-TOC) and [Aire](https://github.com/mr-kelley/aire). Project Prestidigitonium composes with both but requires neither — it is additive at a layer they do not occupy.

## How adoption works (v0.2.0)

1. You have a role file. It declares operating rules (or verification checks, or normative requirements).
2. You read `adoption-guide.md`. Seven steps.
3. You copy the skeleton from `template-auditor-role-file.md` into a new file at `<audited-role-slug>-auditor.role.md`.
4. You fill in the placeholders: `audits:` frontmatter pointer, upstream governance enumeration, asymmetric corpus paths, key-check categories, per-criterion audit interpretations, cross-rule obligations.
5. You add `audited_by: <audited-role-slug>-auditor.role.md` to the audited role file's frontmatter (the only required edit to that file).
6. You create an `audit-corpus/` directory tree at the project root. Empty is fine; v0.1.0-empty is a conformant state.
7. You bump both files' versions and update their provenance.
8. You run a conformance check against `conformance-criteria.md`.

Nothing executes. No new infrastructure spins up. The pattern is operational the moment the auditor file lands and the corpus tree exists.

## Status and stability

This is **v0.2.0**, status **draft**. The architectural revision from v0.1.x to v0.2.0 was a major inversion. The pattern is internally consistent and operationally complete — it can be adopted now — but corpus-population conventions will sharpen as real instances accumulate findings and post-mortems.

Versioning policy is in `auditor-pattern-spec.md` §"Versioning."

Production adopters: pin to the commit hash you adopted against. v0.x.y is pre-stable; future minor bumps may introduce new integration requirements.

## Branches

- **`main`** — v0.2.0+ separate-file architecture. Canonical direction.
- **`archive/v0.1.x-same-file-architecture`** — v0.1.0 + v0.1.1 same-file architecture, preserved verbatim. Frozen; not updated forward. Adopters who deployed against v0.1.x can pin here.

## Roadmap (informal)

- **v0.2.x** — clarifications, examples, additional drift-catalog templates, real-corpus learnings as they accumulate.
- **v0.3.0** — likely additions: project-level multi-role corpus sharing conventions; explicit cross-corpus pollination protocol; conformance criteria for the specifications themselves (self-conformance, currently reserved).
- **v1.0.0** — declared when at least two independent projects have run the pattern in production for three months and the spec's evolution has stabilized.

## License

Apache License 2.0. See `LICENSE`.

---

*Higitus Figitus Migitus Mum — Prestidigitonium!*
