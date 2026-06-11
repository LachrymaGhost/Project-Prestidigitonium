---
name: conformance-criteria
version: 0.3.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [conformance, audit, governance]
references: auditor-pattern-spec.md, template-auditor-role-file.md, audit-corpus-spec.md
---

# Conformance Criteria

This specification defines how to verify that an implementation of the Project Prestidigitonium auditor pattern (v0.2.0, separate-file architecture) conforms to the pattern. Conformance is binary at the criterion level — a criterion passes or fails — but graded at the implementation level, since partial conformance is operationally meaningful.

The criteria below correspond to the integration requirements stated in `auditor-pattern-spec.md` v0.2.0 §"Integration requirements." Each criterion includes:

- **What is checked** — the conformance condition.
- **How it is checked** — the verification procedure, concrete enough to be applied without interpretation.
- **What counts as failure** — the specific deviation that fails the criterion.

A conformant implementation passes all criteria marked **Required**. **Recommended** criteria are operationally beneficial but optional.

## Criterion C-1 (Required) — Separate auditor role file exists with correct frontmatter

**What is checked.** A file named `<audited-role-slug>-auditor.role.md` exists at the same path as the audited role. Its frontmatter contains `audits: <audited-role-slug>.role.md` and `follows_pattern:` declaring this specification.

**How it is checked.** List the directory containing the audited role file; confirm the auditor file's existence at the canonical name. Parse the auditor file's YAML frontmatter; confirm `audits:` field is present and points to the actual audited role file's filename; confirm `follows_pattern:` field is present and names this spec (form: `Project Prestidigitonium v<X.Y.Z>` or equivalent).

**Failure.** Auditor file missing; auditor file present but at a non-canonical name; `audits:` field absent or pointing to a nonexistent file; `follows_pattern:` field absent.

## Criterion C-2 (Required) — Reciprocal `audited_by:` pointer in audited role file

**What is checked.** The audited role file's frontmatter contains `audited_by: <audited-role-slug>-auditor.role.md`, reciprocal to the auditor file's `audits:` field.

**How it is checked.** Parse the audited role file's YAML frontmatter; confirm `audited_by:` field is present and points to the auditor file.

**Failure.** `audited_by:` field absent; field present but pointing to a nonexistent file or a different file than the one declaring `audits:` back.

## Criterion C-3 (Required) — Auditor file follows base role template structure

**What is checked.** The auditor file contains the standard role-file sections inherited from the base role template: Purpose, Scope (with Covers + Does Not Cover), Normative Requirements, Operational Constraints, Inputs, Outputs, Verification, Operating Rules, Relational Implementation, Escalation & Halt, Change Control (with Provenance). Plus audit-specific sections: Verification of audited role files, Cross-rule audit obligations.

**How it is checked.** Read the auditor file; confirm each required H1 (or H2 where appropriate) heading is present.

**Failure.** Any required section missing; Change Control without Provenance subsection.

## Criterion C-4 (Required) — Authority chain anchored in upstream governance

**What is checked.** The auditor file's Normative Requirements (or equivalent) state explicitly that the authority chain bottoms out in upstream governance, with the audited role's rules themselves auditable against upstream governance.

**How it is checked.** Read the Normative Requirements; confirm explicit upstream-governance anchoring; confirm the auditable-rules-themselves clause.

**Failure.** Authority chain stated as "the audited role's rules" without upstream-governance routing; or upstream-governance routing implicit/abstract without enumerated specs in Inputs.

## Criterion C-5 (Required) — Upstream governance enumerated in Inputs

**What is checked.** The auditor file's Inputs section enumerates specific upstream governance paths (not abstract references).

**How it is checked.** Read the Inputs section's "Upstream governance" subsection; confirm specific path references (e.g., `claude/claude.role.base.md`, `claude/spec-spec.md`).

**Failure.** Upstream governance referenced abstractly ("the governance specs"); or governance section missing from Inputs.

## Criterion C-6 (Required) — Adversarial default stated

**What is checked.** The auditor file's Normative Requirements (or Operating Rules) explicitly state the adversarial default: every conformance check begins with verdict `refuted`, flipping to `confirmed` only on evidence surviving refutation.

**How it is checked.** Read the Normative Requirements and Operating Rules; confirm the "default refuted" statement; sample 3+ per-criterion audit interpretations and confirm refutation-frame phrasing.

**Failure.** Adversarial default not stated; or 30%+ of audit interpretations use verification-frame grammar ("the auditor verifies that X" instead of "the auditor attempts to refute X").

## Criterion C-7 (Required) — Independent re-derivation enumerated in Operating Rules

**What is checked.** The auditor file's Operating Rule 3 (or equivalent) enumerates key-check categories specific to the audited role's domain and requires derive-then-compare on those categories.

**How it is checked.** Read Operating Rule 3; confirm explicit category enumeration; confirm the derive-then-compare requirement is stated.

**Failure.** No key-check categories named; or category enumeration purely abstract ("relevant checks") without concrete naming; or derive-then-compare requirement absent.

## Criterion C-8 (Required) — Authority precedence stated

**What is checked.** The auditor file states the three-level precedence: user > upstream governance > audited role's rules > audit interpretation.

**How it is checked.** Read Normative Requirements + Operating Rules for the precedence statement; confirm all three levels appear.

**Failure.** Any of the three precedence levels missing; or precedence stated in a way that inverts a level (e.g., audit interpretation declared authoritative over audited role's rule body).

## Criterion C-9 (Required) — Scope limits acknowledged

**What is checked.** The auditor file's Scope section (or a referenced location) acknowledges that the pattern does not solve capability blind spots, adversarial-input failures, definitional disputes, sustained-load attention failures, and meta-audit. Reference to `auditor-pattern-spec.md` v0.2.0 §"What this pattern does NOT solve" is acceptable.

**How it is checked.** Read the Scope section; confirm explicit enumeration or referenced acknowledgment.

**Failure.** Scope limits not acknowledged; or acknowledgment undercut elsewhere in the file (e.g., a cross-rule obligation claims coverage of one of the named-as-unsolved areas).

## Criterion C-10 (Required) — Per-criterion audit interpretations cover the audited role's rule set

**What is checked.** The auditor file's "Verification of audited role files" section contains one audit-interpretation clause per check in the audited role's rule structure (Operating Rules, Verification, or Normative Requirements — whichever is the rule unit).

**How it is checked.** Enumerate the audited role's rule units; enumerate the auditor file's audit-interpretation subsections; confirm one-to-one correspondence by check number or identifier.

**Failure.** Any rule unit lacks an audit-interpretation clause; or audit clauses exist for rule units that don't exist in the audited role file (likely a renumbering drift).

> Note: rules legitimately unauditable post-hoc are conformant *with* an audit clause stating "this check is unauditable post-hoc; see Check N for the executable check." Conformance requires the clause's presence, not its content depth.

## Criterion C-11 (Required) — Per-criterion clauses are refutation-framed

**What is checked.** Each audit-interpretation clause is phrased in the refutation frame.

**How it is checked.** Read each audit-interpretation clause; confirm refutation-frame phrasing.

**Failure.** Pervasive verification-frame phrasing (≥30% of clauses use "verifies that" or equivalent). This is the operational expression of Mechanism 3.

## Criterion C-12 (Required) — Per-criterion refutation mechanisms concrete

**What is checked.** Each audit-interpretation clause states a refutation mechanism concrete enough that two independent auditors would converge on the same procedure.

**How it is checked.** Sample three clauses; confirm mechanisms name a file/path/tool/check, not a vague intent.

**Failure.** Mechanisms stated in vague terms ("the auditor reviews the work," "the auditor uses judgment") without procedural anchor; or mechanisms reference tools or files that don't exist in the project.

## Criterion C-13 (Required) — Cross-rule obligations meet minimum

**What is checked.** The auditor file contains at least three cross-rule audit obligations. Five-plus recommended.

**How it is checked.** Count the numbered (or bulleted) entries in the "Cross-rule audit obligations" subsection.

**Failure.** Fewer than three cross-rule obligations.

## Criterion C-14 (Required) — All six relational primitives implemented through audit lens

**What is checked.** The auditor file's Relational Implementation section implements Frame, Polarity, Trust, Release, Insistence, Completion, each with Behavior, Evidence, Halt — phrased in the audit lens (not as if the auditor were building work directly).

**How it is checked.** Read the Relational Implementation section; confirm all six primitives present with all three subsections each; confirm phrasing reflects audit posture.

**Failure.** Any primitive missing; any primitive lacking Behavior/Evidence/Halt; primitives phrased as if for execution rather than audit.

## Criterion C-15 (Required) — Asymmetric corpus directory tree exists

**What is checked.** The project repository contains an `audit-corpus/` directory (or the project-declared alternative location) with INDEX files seeded for at least Categories A, B, C, and a `README.md` stating access discipline. Category D's directory exists with an explanatory README pointing to the auditor role file. Category E may be omitted at v0.1.0-empty.

**How it is checked.** List the corpus directory; confirm structural presence of category subdirectories and INDEX files; read the `README.md` for the access-discipline statement.

**Failure.** Corpus directory missing entirely; or category subdirectories missing; or access-discipline statement absent from corpus README.

> Note: empty INDEX files and empty category directories are conformant. v0.1.0-empty is a valid state.

## Criterion C-16 (Required) — Audited role's session load list excludes corpus AND auditor file

**What is checked.** The audited role file's session-context configuration does not include any `audit-corpus/` paths AND does not include the auditor role file (`<audited-role-slug>-auditor.role.md`). Under v0.2.0, the auditor file's body is Category D corpus material; loading it breaks the asymmetry.

**How it is checked.** Read the audited role file's session-context section; grep for `audit-corpus` and for the auditor filename; confirm absence from the load list (or presence only with explicit "auditor-only" annotation).

**Failure.** Corpus paths or auditor filename appear on the audited role's load list without auditor-only annotation; or audited role's session-context section is absent and no equivalent discipline is documented elsewhere.

## Criterion C-17 (Required) — No `# §Audit-Variant` section in audited role file

**What is checked.** Under v0.2.0, the audited role file does not contain a `# §Audit-Variant` section. That pattern belongs to v0.1.x; v0.2.0 forecloses it.

**How it is checked.** Read the audited role file; grep for `# §Audit-Variant`; confirm absence.

**Failure.** `# §Audit-Variant` section present in the audited role file. (Adopters migrating from v0.1.x should extract the section's content into the new auditor file per the adoption guide's "Migration from v0.1.x" section, then remove the section from the audited role file.)

## Criterion C-18 (Recommended) — Corpus entries follow frontmatter schema

**What is checked.** If the corpus contains entries (Categories A, B, C, or E), each carries frontmatter conforming to the schema in `audit-corpus-spec.md` §"Entry frontmatter."

**How it is checked.** Sample entries from each populated category; confirm `id`, `category`, `status`, `date`, `applies_to_rules` (where required), `applies_to_role` are present.

**Failure.** Sampled entries missing required frontmatter fields.

## Criterion C-19 (Recommended) — Post-mortem incorporation traceable

**What is checked.** If any Category B post-mortem has `status: incorporated`, the corresponding change to the auditor role file's audit interpretations or cross-rule obligations is locatable by reference.

**How it is checked.** Sample incorporated post-mortems; for each, locate the corresponding clause change in the auditor file's version history or in the post-mortem's "incorporated by" reference.

**Failure.** Incorporated post-mortems exist but the auditor-file changes they drove cannot be located.

## Criterion C-20 (Recommended) — Audit posture declared explicitly

**What is checked.** The auditor file's frontmatter contains an `audit_posture` field with one of the three valid values (`artifact-verdict-only`, `continuous-monitoring`, `hybrid`), and the body contains a corresponding "Audit posture declaration" section naming the chosen posture.

**How it is checked.** Parse the auditor file's YAML frontmatter; confirm `audit_posture` field is present with a valid value. Read the body; confirm the matching declaration section exists and names the same posture.

**Failure.** `audit_posture` field absent; or field present with an invalid value; or body declaration section absent or naming a different posture than the frontmatter declares.

> Note: under v0.2.2, auditors that fail to declare a posture default to `artifact-verdict-only` per the template's graceful-default rule. Absence is therefore not a *Required* defect — the implementation remains operationally usable — but explicit declaration prevents the posture from being ambiguous to downstream readers, and is therefore Recommended.

## Criterion C-21 (Recommended) — Audited-role version pinned and current

**What is checked.** The auditor file's frontmatter contains an `audits_version:` field naming the version of the audited role file its per-criterion audit interpretations were authored against, and that version matches the audited role file's current frontmatter `version`.

**How it is checked.** Parse both files' YAML frontmatter; confirm `audits_version:` is present in the auditor file; compare its value to the audited role file's current `version` field. Read the auditor file's Operational Constraints for the session-start staleness comparison.

**Failure.** `audits_version:` field absent; or field present but mismatched against the audited role's current version with no staleness acknowledgment in the auditor file's most recent provenance entry.

> Note: Recommended rather than Required because the path-level `audits:` pointer (C-1) already establishes the audit relationship. The version pin protects against a failure C-1 cannot see: the audited role bumps its version, renumbers or revises its checks, and the auditor's interpretations silently drift out of correspondence. Without the pin, that drift surfaces only when a full C-10 correspondence review happens to run; with it, every auditor session detects the mismatch at load time.

## Criterion C-22 (Recommended) — Comms channel established per comms-spec

**What is checked.** The project carries the builder↔auditor mail system per `comms-spec.md`: the auditor file's Operational Constraints include the comms duties (session-start read-then-arm, first-boot setup, housekeeping); the builder's session-entry configuration carries the bootstrap pointer (read inbox at session start); and either the `.comms/` tree exists with the activation directive filed in the builder's inbox, or the auditor has not yet had its first boot (pending state — acceptable when the setup duty is present in the auditor file).

**How it is checked.** Read the auditor file's Operational Constraints for the three duties. Grep the builder's CLAUDE.md (or equivalent session-entry configuration) for the inbox pointer. List `.comms/`; if present, confirm the README and both inboxes exist and the activation directive is filed.

**Failure.** Comms duties absent from the auditor file; bootstrap pointer absent (the activation directive would wait in an inbox nothing opens); tree present but malformed (shared mutable mail file instead of message-per-file inboxes; corpus material inside `.comms/`).

> Note: Recommended rather than Required because the pattern functions with owner-relayed verdicts. The channel removes the human relay — verdicts, flags, and remediation acknowledgments travel between the roles directly, with the owner reading rather than carrying.

> Note (multi-file rule sources, v0.2.5): C-21 as written assumes the audited role's rule units live in the pinned file. When the interpretations target rule units in *other* files — module specs under a master role file, for example — the single pin can match while the targeted files drift: the pin's coverage is conventional (it holds only as long as the master's change control bumps on module changes), not mechanical. Auditors in that topology SHOULD enumerate in their staleness-check clause which files and rule-unit sets the interpretations target, and SHOULD treat a targeted file's version change as interpretation staleness even when the master pin matches. Surfaced by the RoleSmith Auditor's first cross-project verdict (2026-06-11, auditing an auditor whose interpretations target engine and persona module specs). A mechanical multi-pin form (`audits_version:` as a map of file → version) is a candidate for v0.3.0.

## Conformance verdict structure

A conformance verdict is one of:

- **CONFORMANT** — all Required criteria pass; Recommended criteria pass or are not applicable.
- **CONFORMANT WITH NOTES** — all Required criteria pass; one or more Recommended criteria fail; notes document the failures.
- **PARTIALLY CONFORMANT** — between one and three Required criteria fail; the implementation is operationally usable but has gaps.
- **NON-CONFORMANT** — four or more Required criteria fail; the implementation does not realize the pattern even if it claims to.

The thresholds are conventions, not absolutes. A single Required failure on a load-bearing criterion (C-1, C-2, C-4, C-6, C-10, C-11, C-16, C-17) is operationally more serious than three failures on procedural criteria. A verdict should explain its grading; categorical labels are summaries, not substitutes for the narrative.

Check-level `inconclusive` results require their own handling in grading. An implementation-level verdict resting on a substantial fraction of inconclusive checks (as a convention, more than a quarter of the checks reviewed) is weak evidence regardless of how few checks failed outright, and SHOULD be graded no higher than CONFORMANT WITH NOTES — with the notes enumerating each inconclusive check and the specific evidence that would resolve it. Inconclusiveness is not failure, but a verdict that cannot establish most of what it set out to establish should not present itself as clean.

## Performing a conformance review

A conformance review proceeds in three passes:

1. **Structural pass** — confirm C-1, C-2, C-3, C-15, C-17 (file existence, reciprocal pointers, section structure, corpus tree, absence of §Audit-Variant). If structural conformance fails, halt and route fixes; remaining criteria depend on structural presence.

2. **Preamble pass** — confirm C-4 through C-9 (authority chain + upstream governance enumeration + adversarial default + re-derivation + authority precedence + scope limits). These are the design-bearing criteria.

3. **Clause pass** — confirm C-10 through C-14, C-16 (per-criterion clauses + relational primitives + audited-role load discipline). Highest-volume criteria; review proceeds by sampling for projects with 10+ checks in the audited role.

Recommended criteria (C-18 through C-22) are reviewed last.

## Self-conformance of this specification

This specification governs auditor-pattern conformance. It does not claim conformance with itself. A future companion specification may define conformance for the specifications themselves; v0.2.0 does not include it.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Major revision per architectural inversion from same-file (v0.1.x) to separate-file (v0.2.0).
- time: 2026-06-07
- summary: v0.2.0 — Rewritten conformance criteria for the separate-file architecture. v0.1.x's sixteen criteria (centered on §Audit-Variant section presence and structure) replaced with nineteen criteria (seventeen Required, two Recommended) centered on separate-file topology: auditor file existence + canonical naming + correct frontmatter (C-1); reciprocal `audited_by:` pointer in audited role (C-2); base-role-template structure in auditor file (C-3); authority chain + upstream governance + adversarial default + re-derivation + authority precedence + scope limits (C-4 to C-9); per-criterion clauses + refutation framing + concrete mechanisms + cross-rule obligations (C-10 to C-13); all six relational primitives through audit lens (C-14); corpus tree (C-15); audited-role load list excludes both corpus AND auditor file (C-16); absence of §Audit-Variant in audited role (C-17); corpus frontmatter discipline + post-mortem incorporation traceability (C-18, C-19 Recommended). Three-pass review procedure updated. Load-bearing criteria called out (C-1, C-2, C-4, C-6, C-10, C-11, C-16, C-17). v0.2.2 (2026-06-07) — Adds Criterion C-20 (Recommended) for explicit `audit_posture` declaration in auditor file frontmatter and body, per the template's new posture-declaration mechanism (artifact-verdict-only / continuous-monitoring / hybrid). Absence is not a Required defect because v0.2.2's template specifies a graceful default to `artifact-verdict-only`, but explicit declaration prevents downstream ambiguity. References field updated to v0.2.2 across all three sibling specs. No Required criteria changed; v0.2.0-conformant implementations remain conformant under v0.2.2. v0.2.3 (2026-06-11) — Per ASA fresh-pass review: restores numerical ordering of the Recommended criteria (C-20 had been filed before C-19 in v0.2.2) and adds C-20/C-21 to the review-procedure closing line, which still read "C-18, C-19" after C-20 landed. Adds Criterion C-21 (Recommended) for `audits_version:` interpretation pinning — the auditor declares which version of the audited role file its per-criterion interpretations target, detecting silent interpretation drift at session start rather than waiting for a full C-10 correspondence review. Adds inconclusive-density guidance to the verdict structure: verdicts resting on more than roughly a quarter inconclusive checks SHOULD grade no higher than CONFORMANT WITH NOTES. Follows the C-20 precedent of landing Recommended criteria in a patch release; no Required criteria changed; v0.2.0-conformant implementations remain conformant under v0.2.3. v0.2.5 (2026-06-11) — C-21 gains the multi-file rule-source note per the RoleSmith Auditor's first cross-project verdict: when interpretations target rule units in files other than the pinned master (module specs), the single pin's coverage is conventional rather than mechanical; auditors in that topology SHOULD enumerate targeted files in their staleness clause and treat targeted-file version changes as staleness. Multi-pin map form deferred to v0.3.0. No criteria added or changed; guidance note only. v0.2.6 (2026-06-11) — References field converted to pin-free (filenames only; versions live in each file and the CHANGELOG) per the drift catalog's intervention; version aligned to the release lockstep. No criteria changes. v0.3.0 (2026-06-11) — Adds Criterion C-22 (Recommended): comms channel established per `comms-spec.md` (auditor file carries the read-then-arm, setup, and housekeeping duties; builder's session-entry configuration carries the bootstrap pointer; tree well-formed when present, with first-boot-pending accepted). Recommended because the pattern functions with owner-relayed verdicts; the channel removes the human relay. Review-procedure closing line extended through C-22.
