# Project Prestidigitonium

> A domain-agnostic auditor pattern for role-based AI governance systems.

Project Prestidigitonium is a plug-and-play specification for adding an **auditor** alongside any role in a role-based AI governance system — without inheriting the role's blind spots, and without losing the domain fluency that makes audits substantive rather than merely procedural.

The pattern is designed to drop into existing role files (Aire-shaped, or any equivalent with explicit operating rules) by way of a single `# §Audit-Variant` section. No new files. No new infrastructure. The audit lens lives in the same role spec the builder lives in — same surface, different authority chain underneath.

## What's in this repo

| File | Purpose |
|---|---|
| `auditor-pattern-spec.md` | The canonical specification. Encompassment principle + four mechanisms (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation) + naming, authority, scope limits, integration requirements. |
| `template-audit-variant-section.md` | Copy-pasteable skeleton for the `# §Audit-Variant` section. Annotated companion explaining what each preamble element is for. |
| `audit-corpus-spec.md` | Structure, growth rules, and access discipline for the asymmetric corpus — the body of materials the auditor loads but the builder does not. |
| `adoption-guide.md` | Seven-step walkthrough from "role file with rules" to "conformant §Audit-Variant section landed." |
| `conformance-criteria.md` | Sixteen criteria for verifying that an implementation realizes the pattern. Includes verdict structure and review procedure. |
| `CHANGELOG.md` | Version history. |
| `LICENSE` | Apache 2.0. |

## The problem this solves

A role file specifies how a builder should reason about its domain. If a defect lives inside that specification — an unstated assumption, a missing trigger, a conflated category — a reader who treats the specification as authoritative cannot see the defect. The defect is invisible *because* it is the lens through which the reader perceives. Adding an "auditor lens" that reads the same specification with a different intent does not solve this: same words cannot catch what the words do not say.

Naive auditor designs sit at one of two failure modes:

- **Same-file lens** — auditor and builder read the same rules. Domain fluency preserved; correlated blind spots inherited.
- **Fully-independent reviewer** — auditor knows nothing about the role. Blind spots avoided; domain fluency lost.

Project Prestidigitonium resolves the tension. Surface identity, underlying independence: same role file invocation; encompassing scope; asymmetric corpus; adversarial reading posture; independent re-derivation on key checks.

## Key terms (glossary)

- **Builder** — the role being audited. The thing that *does the work*. (In Aire shape: typically named `<project>-<role>` like `sketch-operator` or `aire-smith`.)
- **Auditor** — the audit-lens reading of the same role file. Named `<Project> <Role> Auditor`. The thing that *checks the work*.
- **Encompassment** — the structural property that the auditor's knowledge is a proper superset of the builder's: `auditor_knowledge ⊇ builder_knowledge` with `auditor_knowledge \ builder_knowledge ≠ ∅`.
- **Asymmetric corpus** — materials the auditor loads that the builder does not. Prior findings, failure post-mortems, drift catalogs, the §Audit-Variant clauses themselves, and project-specific asymmetric materials.
- **Adversarial default** — the auditor's epistemic stance. Default verdict: refuted; flips to confirmed only on evidence surviving refutation.
- **Independent re-derivation** — derive-then-compare on key checks. Reading the builder's output before deriving is not permitted on these checks.
- **§Audit-Variant** — the section in the role file where the audit lens is encoded. One audit-interpretation clause per rule plus cross-rule obligations.
- **Supervisory layer** — the asymmetric portion of the auditor's knowledge: upstream governance authority + asymmetric corpus + adversarial posture + re-derivation discipline. What makes the auditor *more than* a different reading of the same words.

## Why a separate repository

This specification is governance-pattern material, not project-specific code. It is published as a standalone repository so that:

- It can evolve independently of any one project's release cadence.
- Projects can pin to a specific spec version and upgrade deliberately.
- Adopters across organizations can reference the same canonical pattern.
- It can be cited in role-file provenance as a stable artifact.

This follows the precedent of [Aire-TOC](https://github.com/LachrymaGhost/Aire-TOC) (the context-routing pattern published separately from any single implementation) and [Aire](https://github.com/mr-kelley/aire) (the governance framework published as its own repo). Project Prestidigitonium composes with both but requires neither — it is additive at a layer they do not occupy.

## How adoption works

1. You have a role file. It declares operating rules (or verification checks, or normative requirements).
2. You read `adoption-guide.md`. Seven steps.
3. You copy the skeleton from `template-audit-variant-section.md` into your role file at the correct structural position.
4. You fill in the placeholders: encompassment preamble, asymmetric corpus paths, adversarial-default statement, independent-re-derivation key-check categories, authority precedence, per-rule audit clauses, cross-rule obligations.
5. You create an `audit-corpus/` directory tree at the project root. Empty is fine; v0.1.0-empty is a conformant state.
6. You bump the role file's version and update its provenance.
7. You run a conformance check against `conformance-criteria.md`.

Nothing executes. No new infrastructure spins up. The pattern is operational the moment the section lands and the corpus tree exists.

## Status and stability

This is **v0.1.0**, status **draft**. The pattern is internally consistent and operationally complete — it can be adopted now — but the corpus-population conventions (Categories A through E) will sharpen as real instances accumulate findings and post-mortems. Versioning policy is in `auditor-pattern-spec.md` §"Versioning"; breaking changes warrant a major bump and will be telegraphed in `CHANGELOG.md`.

Production adopters: pin to the commit hash you adopted against. v0.x.y is pre-stable; minor bumps may introduce new integration requirements.

## Roadmap (informal)

- **v0.1.x** — clarifications, examples, additional drift-catalog templates.
- **v0.2.0** — likely additions: project-level multi-role corpus sharing conventions; explicit cross-corpus pollination protocol; conformance criteria for the specifications themselves (self-conformance, currently reserved).
- **v1.0.0** — declared when at least two independent projects have run the pattern in production for three months and the spec's evolution has stabilized.

## License

Apache License 2.0. See `LICENSE`.

---

*Higitus Figitus Migitus Mum — Prestidigitonium!*
