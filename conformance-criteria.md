---
name: conformance-criteria
version: 0.1.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [conformance, audit, governance]
references: auditor-pattern-spec.md v0.1.0, template-audit-variant-section.md v0.1.0, audit-corpus-spec.md v0.1.0
---

# Conformance Criteria

This specification defines how to verify that an implementation of the Project Prestidigitonium auditor pattern conforms to the pattern. Conformance is binary at the criterion level — a criterion passes or fails — but graded at the implementation level, since partial conformance is operationally meaningful: an implementation may be conformant at v0.1.0-empty corpus state and grow toward fuller conformance as the corpus matures.

The criteria below correspond to the integration requirements stated in `auditor-pattern-spec.md` §"Integration requirements." Each criterion includes:

- **What is checked** — the conformance condition.
- **How it is checked** — the verification procedure, concrete enough to be applied without interpretation.
- **What counts as failure** — the specific deviation that fails the criterion.

A conformant implementation passes all criteria marked **Required**. **Recommended** criteria are operationally beneficial but optional; absence is not a conformance failure.

## Criterion C-1 (Required) — §Audit-Variant section exists

**What is checked.** The role file contains a top-level `# §Audit-Variant` heading at the correct structural position (under `# Change Control` → `## Provenance`, as a peer of the role's other top-level sections).

**How it is checked.** Read the role file; confirm a heading matching the regex `^# §Audit-Variant — ` exists; confirm `## Provenance` appears under `# Change Control` *before* the §Audit-Variant heading (preserving Provenance as a child of Change Control and §Audit-Variant as a top-level peer).

**Failure.** §Audit-Variant heading missing; or §Audit-Variant exists but with wrong heading level (e.g., `## §Audit-Variant`); or Provenance falls structurally under §Audit-Variant instead of under Change Control.

## Criterion C-2 (Required) — Heading follows the naming convention

**What is checked.** The §Audit-Variant heading is in the form `# §Audit-Variant — <Project> <Role> Auditor's lens on the <N> <rule-unit-name>`.

**How it is checked.** Read the heading; confirm the format matches; confirm `<N>` is a number; confirm `<rule-unit-name>` corresponds to an actual rule-unit structure in the role file (Operating Rules, Verification, etc.).

**Failure.** Heading uses metaphor language (foreman, supervisor, twin, etc.); or `<Project>` or `<Role>` not stated; or `<N>` is non-numeric or absent; or `<rule-unit-name>` does not correspond to a real rule unit in the file.

## Criterion C-3 (Required) — Encompassment preamble present and complete

**What is checked.** The §Audit-Variant section begins with a preamble paragraph operationalizing Mechanism 1 — explicitly stating that the auditor's knowledge is a proper superset of the builder's, enumerating upstream governance sources, and stating that the role file's rules are themselves auditable.

**How it is checked.** Read the first paragraph(s) of the §Audit-Variant section; confirm all three elements appear; confirm upstream governance is enumerated (not merely referenced abstractly).

**Failure.** Encompassment principle not stated; or upstream governance enumerated only abstractly ("the governance specs") without specific paths; or absence of the "role file's rules are themselves auditable" clause, which is the load-bearing escape hatch from same-source blind spots.

## Criterion C-4 (Required) — Asymmetric corpus access stated

**What is checked.** The §Audit-Variant section contains an asymmetric-corpus-access paragraph operationalizing Mechanism 2 — listing the corpus sources the auditor loads, stating that the builder MUST NOT load them by default, and including the §Audit-Variant clauses themselves as a self-referential corpus entry (Category D per `audit-corpus-spec.md`).

**How it is checked.** Read the asymmetric-corpus paragraph; confirm corpus paths are stated by path or by stated convention; confirm the MUST-NOT-load discipline appears; confirm Category D inclusion is acknowledged.

**Failure.** Corpus sources not stated; or "MUST NOT load" discipline absent; or Category D not acknowledged. (Acknowledgment is required; population may be deferred — Category D's content lives in the §Audit-Variant clauses themselves, so it is non-empty by construction once the clauses are authored.)

## Criterion C-5 (Required) — Adversarial default stated

**What is checked.** The §Audit-Variant section contains an adversarial-default paragraph operationalizing Mechanism 3 — stating that the default verdict is **refuted** and that verdicts flip to **confirmed** only on evidence surviving refutation attempts.

**How it is checked.** Read the adversarial-default paragraph; confirm the explicit "default verdict: refuted" statement; confirm the audit-clause grammar below adheres to the refutation frame (sampling 3+ clauses).

**Failure.** Default-refuted statement absent; or audit clauses written in the verification frame ("the auditor verifies that X") instead of the refutation frame ("the auditor attempts to refute X") for the sampled clauses.

> Note: a partial drift toward verification-frame phrasing in one or two clauses is a soft failure (correctable in the next revision); pervasive verification-frame phrasing across the section is a hard failure (the mechanism is not operational).

## Criterion C-6 (Required) — Independent re-derivation enumerated

**What is checked.** The §Audit-Variant section contains an independent-re-derivation paragraph operationalizing Mechanism 4 — enumerating key-check categories for this role and stating that derive-then-compare is required for these categories.

**How it is checked.** Read the independent-re-derivation paragraph; confirm the enumeration is concrete (not "as applicable" or "whatever the auditor judges relevant"); confirm at least one category is named.

**Failure.** No key-check categories named; or category enumeration is purely abstract ("relevant checks") without concrete naming; or absence of the derive-then-compare requirement statement.

## Criterion C-7 (Required) — Authority precedence stated

**What is checked.** The §Audit-Variant section states the three-level precedence: execute-mode rule body authoritative over audit-interpretation clause on lens conflicts; upstream governance authoritative over role file rules; user authoritative over auditor verdicts.

**How it is checked.** Read the authority-precedence paragraph; confirm all three precedence levels appear.

**Failure.** Any of the three precedence levels missing; or precedence stated in a way that inverts one of the three (e.g., audit-interpretation declared authoritative over execute-mode rule body).

## Criterion C-8 (Required) — Scope limits acknowledged

**What is checked.** The §Audit-Variant section includes an acknowledgment that the pattern does not solve capability blind spots, adversarial-input failures, definitional disputes about what constitutes a defect, or sustained-load attention failures. Reference to `auditor-pattern-spec.md` §"What this pattern does NOT solve" is acceptable in lieu of enumeration.

**How it is checked.** Read the scope-limits paragraph or section; confirm either explicit enumeration or referenced acknowledgment.

**Failure.** Scope limits not acknowledged; or acknowledgment is undercut elsewhere in the section (e.g., a cross-rule obligation claims coverage of one of the named-as-unsolved areas).

## Criterion C-9 (Required) — Per-rule audit clauses total

**What is checked.** Every operating rule (or verification check, or normative requirement — whichever rule-unit the §Audit-Variant heading declared) has a corresponding audit-interpretation clause in the section.

**How it is checked.** Enumerate the role file's rule units; enumerate the audit-interpretation subsections; confirm one-to-one correspondence by rule number or rule identifier.

**Failure.** Any rule unit lacks an audit-interpretation clause; or the section contains audit clauses for rule units that don't exist in the role file (likely indicating a renumbering drift that needs resolution).

> Note: rules legitimately unauditable post-hoc are conformant *with* an audit clause stating "this rule is unauditable post-hoc; see Rule N for the executable check." Conformance requires the clause's presence, not its content depth.

## Criterion C-10 (Required) — Per-rule clauses are refutation-framed

**What is checked.** Each audit-interpretation clause is phrased in the refutation frame: "the auditor attempts to refute the claim that X" rather than "the auditor verifies that X."

**How it is checked.** Read each audit-interpretation clause; confirm refutation-frame phrasing.

**Failure.** Pervasive verification-frame phrasing (≥30% of clauses use "verifies that" or equivalent confirmation grammar). This is the operational expression of Mechanism 3; failure here neutralizes the adversarial default.

## Criterion C-11 (Required) — Per-rule refutation mechanisms concrete

**What is checked.** Each audit-interpretation clause states a refutation mechanism concrete enough that two independent auditors would converge on the same procedure.

**How it is checked.** Read each clause; confirm the mechanism names a file/path/tool/check, not a vague intent. Sample three clauses; if mechanisms are concrete in all three, the criterion passes.

**Failure.** Mechanisms stated in vague terms ("the auditor reviews the work," "the auditor uses judgment") without procedural anchor; or mechanisms reference tools or files that don't exist in the project.

## Criterion C-12 (Required) — Cross-rule obligations meet minimum

**What is checked.** The §Audit-Variant section contains at least three cross-rule audit obligations. Five recommended.

**How it is checked.** Count the numbered (or bulleted) entries in the cross-rule obligations subsection.

**Failure.** Fewer than three cross-rule obligations.

## Criterion C-13 (Required) — Asymmetric corpus directory tree exists

**What is checked.** The project repository contains an `audit-corpus/` directory (or the project-declared alternative location) with INDEX files seeded for at least Categories A, B, C, and a `README.md` stating access discipline. Category D's directory exists with an explanatory README pointing to the §Audit-Variant section in the role file. Category E may be omitted at v0.1.0-empty.

**How it is checked.** List the corpus directory; confirm the structural presence of category subdirectories and INDEX files; read the `README.md` for the access-discipline statement.

**Failure.** Corpus directory missing entirely; or category subdirectories missing; or access-discipline statement absent from corpus README.

> Note: empty INDEX files and empty category directories are conformant. v0.1.0-empty is a valid state; the criterion checks for structural presence, not population.

## Criterion C-14 (Required) — Builder load list excludes corpus

**What is checked.** The role file's session-context configuration (typically the "Session-start load list" or equivalent) does not include any `audit-corpus/` paths.

**How it is checked.** Read the role file's session-context section; grep for `audit-corpus`; confirm absence from the load list, or presence only with explicit "auditor-only" annotation.

**Failure.** Corpus paths appear on the builder's load list without auditor-only annotation; or the role file's session-context section is absent and no equivalent discipline is documented elsewhere.

## Criterion C-15 (Recommended) — Corpus entries follow frontmatter schema

**What is checked.** If the corpus contains any entries (Categories A, B, C, or E), each entry carries frontmatter conforming to the schema in `audit-corpus-spec.md` §"Entry frontmatter."

**How it is checked.** Sample entries from each populated category; confirm `id`, `category`, `status`, `date`, `applies_to_rules` (where required), `applies_to_role` are present.

**Failure.** Sampled entries missing required frontmatter fields, or fields populated with non-conforming values.

## Criterion C-16 (Recommended) — Post-mortem incorporation traceable

**What is checked.** If any Category B post-mortem has `status: incorporated`, the corresponding change to the §Audit-Variant clauses or cross-rule obligations is locatable by reference.

**How it is checked.** Sample incorporated post-mortems; for each, locate the corresponding clause change in the role file's version history or in the post-mortem's "incorporated by" reference.

**Failure.** Incorporated post-mortems exist but the role file changes they drove cannot be located.

## Conformance verdict structure

A conformance verdict is one of:

- **CONFORMANT** — all Required criteria pass; Recommended criteria pass or are not applicable.
- **CONFORMANT WITH NOTES** — all Required criteria pass; one or more Recommended criteria fail; notes document the failures.
- **PARTIALLY CONFORMANT** — between one and three Required criteria fail; the implementation is operationally usable but has gaps that should be closed.
- **NON-CONFORMANT** — four or more Required criteria fail; the implementation does not realize the pattern even if it claims to.

The thresholds are conventions, not absolutes. A single Required failure on a load-bearing criterion (C-1, C-3, C-4, C-5, C-9, C-10) is operationally more serious than three Required failures on procedural criteria (C-7, C-8, C-13). A verdict should explain its grading; categorical labels are summaries, not substitutes for the narrative.

## Performing a conformance review

A conformance review proceeds in three passes:

1. **Structural pass** — confirm C-1, C-2, C-13 (heading, naming, corpus tree). If structural conformance fails, halt the review and route fixes back to the adopter; remaining criteria depend on structural presence.

2. **Preamble pass** — confirm C-3 through C-8 (the five mechanism preambles + authority + scope limits). These are the design-bearing criteria; failure here means the implementation has wrong-shape independence even if the clauses look conformant.

3. **Clause pass** — confirm C-9, C-10, C-11, C-12 (per-rule clauses + cross-rule obligations). These are the highest-volume criteria; review proceeds by sampling rather than exhaustive read for projects with 10+ rules.

Recommended criteria (C-15, C-16) are reviewed last and produce notes rather than blocking findings.

## Self-conformance of this specification

This specification governs auditor-pattern conformance. It does not claim conformance with itself — it is a specification, not a role file. A future companion specification (or this one extended) may define conformance for the specifications themselves (consistency of cross-references, version coherence across the three core specs, etc.); v0.1.0 does not include it.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Initial draft.
- time: 2026-06-06
- summary: v0.1.0 — Initial conformance criteria. Sixteen criteria covering the integration requirements stated in auditor-pattern-spec.md §"Integration requirements." Required vs. Recommended grading; four-level verdict structure (CONFORMANT / CONFORMANT WITH NOTES / PARTIALLY CONFORMANT / NON-CONFORMANT); three-pass review procedure (structural / preamble / clause). Explicit note that the specification does not claim conformance with itself; reserved for a future companion specification.
