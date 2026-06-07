# Changelog

All notable changes to Project Prestidigitonium are documented here. Versioning follows semantic versioning per `auditor-pattern-spec.md` §"Versioning."

## [0.2.1] — 2026-06-07

### Added
- `auditor-pattern-spec.md` v0.2.1 — Explicit supersession of v0.1.x's "no separate file" architectural language. Quotes the v0.1.x template's note ("The role IS the capability; `audit` is one of two lenses... there is no separate `*-auditor.role.md` file") and the v0.1.x adoption-guide statement that adopters wondering if they needed one "do not." States the reversal under v0.2.0+ so adopters pinned to v0.1.x reading the confident "no" understand the project itself has changed direction.
- `auditor-pattern-spec.md` §"The shift in plain terms" — articulates *why* the architectural inversion is an improvement rather than a reorganization: v0.1.x asymmetry was enforced by load-discipline (reminder, not gate); v0.2.0 asymmetry is enforced by the file boundary itself (structural, not declarative). Framing credited to Sketch Main Auditor's observation.
- `adoption-guide.md` v0.2.1 — New "Project CLAUDE.md routing under separate-file architecture" section documenting three routing strategies (Strategy A: separate CLAUDE.md per role/auditor directory, recommended; Strategy B: single CLAUDE.md with explicit instance dispatch; Strategy C: auditor invocation bypasses CLAUDE.md auto-load). Addresses the salience-contamination risk when a hardcoded CLAUDE.md routes both builder and auditor invocations to the audited role's content.

### Origin
Both additions originated from Sketch Main Auditor's review of the archive branch + main bookmark before v0.2.0 was pushed. Their observations: (1) the v0.1.x confident "no" needed explicit forward supersession in v0.2.0; (2) CLAUDE.md routing under v0.2.0 deserved adoption guidance, not just structural-requirement implication. Both incorporated.

### Conformance
v0.2.1 is editorial relative to v0.2.0 — no integration-requirement changes; no conformance-criteria changes. v0.2.0 implementations remain conformant under v0.2.1 without modification.

## [0.2.0] — 2026-06-07

### Changed (BREAKING — major architectural revision)
- **Architecture inverted from same-file to separate-file.** v0.1.x committed to a `# §Audit-Variant` section in the audited role's file. v0.2.0 inverts to a separate `<audited-role-slug>-auditor.role.md` file with the audited role declared as an Input via reciprocal `audits:` / `audited_by:` frontmatter pointers. Reasoning: same-file preserved domain fluency at the cost of shared source text — the auditor reading the same words could not catch what those words did not say. Mechanism-level mitigations (asymmetric corpus, adversarial frame, independent re-derivation) addressed the symptom; separate-file addresses the structural cause.
- **The four mechanisms survive the inversion** (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation). Their operationalization moves from "section in the audited role file" to "structure of the separate auditor file."

### Removed
- `template-audit-variant-section.md` (v0.1.x). Replaced by `template-auditor-role-file.md`.
- The `# §Audit-Variant` section pattern in audited role files. Audited role files contain no audit lens content under v0.2.0; the auditor lives in its own file. Adopters migrating from v0.1.x extract the section content into the new auditor file per `adoption-guide.md` §"Migration from v0.1.x."

### Added
- `template-auditor-role-file.md` v0.2.0 — Full auditor role file skeleton (frontmatter through Change Control), replacing the v0.1.x section template.
- Conformance criteria C-1 (separate auditor role file exists with correct frontmatter), C-2 (reciprocal `audited_by:` pointer), C-17 (no `# §Audit-Variant` section in audited role file).
- `adoption-guide.md` §"Migration from v0.1.x" — five-step migration path for v0.1.x adopters.
- `auditor-pattern-spec.md` §"File-relationship topology" — declares the audited / auditor / governance / corpus file relationships explicitly.

### Branch policy
- `main` tracks v0.2.0+.
- `archive/v0.1.x-same-file-architecture` preserves v0.1.0 + v0.1.1 verbatim. Adopters who deployed against v0.1.x can pin to this branch.

## [0.1.1] — 2026-06-06

### Added
- `audit-corpus-spec.md` v0.1.1 — Active-learning hooks: documents machine-readable frontmatter surfaces, detection-layer signal types, promotion-layer hooks, evolution-layer caution, and adopter discipline that produces active-learning-ready corpora today. Adds the `informed` field to the entry frontmatter schema as the cross-category linkage signal (Category A → Category C promotion provenance). Forward-compatible patch; v0.1.0 implementations remain conformant.

### Deferred
- Active-learning extension specification (anticipated `active-learning-spec.md`). Authored when accumulated real corpora provide evidence to design against rather than designing in the abstract.

## [0.1.0] — 2026-06-06

### Added
- `auditor-pattern-spec.md` v0.1.0 — canonical specification: encompassment principle; four mechanisms (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation); naming convention (`<Project> <Role> Auditor`); three-level authority precedence; scope-limit acknowledgments; integration requirements; semantic versioning policy.
- `template-audit-variant-section.md` v0.1.0 — copy-pasteable `# §Audit-Variant` skeleton with annotated companion explaining each preamble element, per-rule clause structure, and cross-rule obligation set.
- `audit-corpus-spec.md` v0.1.0 — structure, growth rules, and access discipline for the asymmetric corpus: five entry categories (prior findings, failure post-mortems, drift catalogs, self-referential §Audit-Variant clauses, project-specific materials); default disk layout; entry frontmatter schema; bootstrapping guidance for v0.1.0-empty corpora; anti-pattern catalog.
- `adoption-guide.md` v0.1.0 — seven-step walkthrough for instantiating the pattern in an existing role file; what-changes-for-builder and what-changes-for-auditor sections; recurring adoption failure modes; compact adoption checklist.
- `conformance-criteria.md` v0.1.0 — sixteen conformance criteria (fourteen Required, two Recommended); four-level verdict structure (CONFORMANT / CONFORMANT WITH NOTES / PARTIALLY CONFORMANT / NON-CONFORMANT); three-pass review procedure (structural / preamble / clause).
- `README.md` — front-door overview; glossary of load-bearing terms; rationale for separate repository; adoption summary; roadmap.
- `LICENSE` — Apache 2.0 (carried from initial repository creation).
