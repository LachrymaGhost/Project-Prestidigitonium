---
name: adoption-guide
version: 0.2.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [guide, adoption, audit, governance]
references: auditor-pattern-spec.md v0.2.0, template-auditor-role-file.md v0.2.0, audit-corpus-spec.md v0.2.0
---

# Adoption Guide

This guide walks an adopter through instantiating the Project Prestidigitonium auditor pattern in an existing role-based AI governance system under v0.2.0's separate-file architecture. The adopter creates a new auditor role file alongside the existing audited role file; the audited role file gets a single frontmatter pointer added but is otherwise unchanged.

This is the practical companion to `auditor-pattern-spec.md` (which states the pattern in principle), `template-auditor-role-file.md` (which provides the copy-pasteable skeleton), and `audit-corpus-spec.md` (which defines the asymmetric corpus).

The guide assumes you have:

- A role file conforming to a base role template (Aire's `claude.role.base.md` or equivalent).
- An understanding of the role's domain sufficient to author per-criterion audit interpretations.
- A repository in which the role file and associated governance specs live.

The guide does **not** assume a multi-role project, sprint discipline, existing audit history, or pre-existing failure-pattern documentation. The pattern adopts cleanly into a single-role project from scratch.

## Adoption in seven steps

### Step 1 — Identify the audited role and its verification structure

Locate the role file the auditor will audit. Identify the rule unit the per-criterion audit interpretations will attach to. Almost every Aire-shaped role has one of:

- A numbered **Operating Rules** section (e.g., Rules 1–13).
- A numbered **Verification** section (e.g., Checks V1–V12).
- A bulleted **Normative Requirements** section (each MUST as the unit).

The structure determines what the per-criterion clauses in the auditor file align against. If multiple structures exist, choose the most operationally consequential.

If the audited role has no rule structure — only prose — adopting the pattern requires first imposing rule structure. The pattern does not work on prose-only roles.

### Step 2 — Choose the auditor's filename and display name

**Filename** (per `auditor-pattern-spec.md` naming convention): `<audited-role-slug>-auditor.role.md`, placed at the same path as the audited role.

**Display name**: `<Project> <Role> Auditor`.

Examples:
- Audited role `aire-smith.role.md` → auditor file `aire-smith-auditor.role.md`; display name "RoleSmith Auditor".
- Audited role `sketch-operator.role.md` → auditor file `sketch-operator-auditor.role.md`; display name "Sketch Operator Auditor".
- Audited role `sketch.role.md` (a Sketch Builder) → auditor file `sketch-auditor.role.md`; display name "Sketch Builder Auditor".

Avoid metaphor language in the artifact (no foreman/boss, no twin-android language). Metaphors are fine for design discussion; the artifact uses functional naming.

### Step 3 — Identify upstream governance

Enumerate the specs that form the auditor's authority bedrock. These are the documents that say "what compliance means" independently of how the audited role file interprets it. Typical sources:

- The base role template (`claude/claude.role.base.md` or project equivalent).
- Kit specs the audited role references.
- The foundations or governance-floor spec for the project.
- Any project-level brief or contract.
- External standards or licenses.

These get listed in the auditor file's Inputs section under "Upstream governance." They are the answer to "if the audited role file is wrong, what is right?"

### Step 4 — Decide the asymmetric corpus location

Per `audit-corpus-spec.md`, the corpus lives in a directory tree. Default: `audit-corpus/` at the project root. Multi-role projects may share a single corpus or maintain per-role corpora (`audit-corpus/<role-slug>/`).

At adoption time, create the directory tree with empty INDEX files and a brief `README.md` stating the access discipline. The corpus may be v0.1.0-empty — the pattern operates with no entries — and grows from real use.

### Step 5 — Enumerate key-check categories

Independent re-derivation (Mechanism 4) is role-specific. Determine which checks must be derive-then-compare in the auditor's Operating Rule 3. Common categories:

- **Structural** — section presence, frontmatter completeness, relational-primitive enumeration (universal for role-file audits).
- **Governance embedding** — derive each MUST from its governance spec; check artifact's embed.
- **Reference resolution** — derive cited paths; check each exists.
- **Re-measurement** — for roles producing numeric outputs.
- **Re-classification** — for roles producing categorical outputs.
- **Re-derivation of acceptance criteria** — for roles operating under written briefs.
- **Rule-applicability mapping** — for roles where which rules govern is itself contested.

The auditor file enumerates the categories applicable to its specific audited role.

### Step 6 — Author the auditor role file

Copy the skeleton from `template-auditor-role-file.md` into a new file at the path determined in Step 2. Fill in:

**Frontmatter:**
- `role: <Project> <Role> Auditor` (display name)
- `version: 0.1.0`
- `audits: <audited-role-slug>.role.md`
- `follows_pattern: Project Prestidigitonium v0.2.0`
- Other standard frontmatter (actor, platform, maintained_by, domain_tags, status, license)

**Body sections** (the template provides the structure):
- Purpose, Scope (Covers + Does Not Cover)
- Normative Requirements (the auditor's own MUSTs — adversarial default, derive-then-compare, citation discipline, authority precedence, falsifiable-correctness, no-modification, etc.)
- Operational Constraints (output format, session-start load discipline, cold-context posture)
- Inputs (audit subject, audited role file per `audits:`, upstream governance from Step 3, asymmetric corpus from Step 4, pattern reference)
- Outputs (verdict + findings + provenance)
- Verification (how the auditor's own work is verified)
- Operating Rules (Rules 1–7 from template: audited-artifact priority, refutation frame, derive-then-compare with categories from Step 5, citation discipline, asymmetric corpus consultation, authority precedence, falsifiable-correctness)
- Verification of audited role files — one audit interpretation per check in the audited role's rule structure (from Step 1)
- Cross-rule audit obligations (minimum three; five-plus recommended)
- Relational Implementation (all six primitives through the audit lens)
- Escalation & Halt Conditions
- Change Control + Provenance

For each per-criterion audit interpretation, author:
- **What the auditor attempts to refute.** Phrase as a refutable proposition (refutation frame is load-bearing).
- **Refutation mechanism.** Concrete enough that two independent auditors would converge.
- **Audit fail conditions.** When the refutation succeeds.

### Step 7 — Add reciprocal pointer to the audited role file, bump versions, surface

Edit the audited role's frontmatter to add: `audited_by: <audited-role-slug>-auditor.role.md`. This is the only required edit to the audited role file under v0.2.0. (Optional: scrub any deprecated language, update governance references — but those are independent housekeeping, not adoption requirements.)

Bump the audited role's version per the project's versioning conventions (minor bump typical: it gained a frontmatter pointer and an audit relationship).

Update both files' Provenance subsections with summaries noting:
- The new auditor file's existence.
- Reciprocal pointer added to the audited role.
- Conformance with `auditor-pattern-spec.md` v0.2.0.
- The asymmetric corpus location.

Surface the change to maintainers and to whoever runs audit passes. The pattern is operational the moment the auditor file lands and the corpus tree exists.

## What changes for the audited role's builder

Almost nothing. The builder reads its own role file as before, with the same operating rules, the same governance compliance, the same outputs. The only changes:

1. **A new frontmatter field** (`audited_by:`) — informational only; does not change behavior.
2. **The audit corpus exists.** The builder MUST NOT load `audit-corpus/` paths or the auditor role file at session start. The role's session-context-load list (if explicitly enumerated) excludes both.
3. **Findings may surface.** When the auditor runs, findings route to the user (or to the project's findings channel). These behave like any other governance flag.

## What changes for the auditor

Everything that wasn't already there. The auditor:

1. Loads its own role file.
2. Loads the audited role file (per `audits:` frontmatter) for domain fluency.
3. Loads upstream governance.
4. Loads the asymmetric corpus's active entries.
5. Loads project state (brief, contract, STATE.md if present).
6. Does NOT load the audited role's auto-memory or session narrative — sees the artifact fresh.
7. Applies the adversarial default to each conformance check.
8. Performs derive-then-compare on the enumerated key-check categories.
9. Cites corpus entries when applying their patterns; cites upstream governance when overriding the audited role's rule; cites the user when deferring to a directive.
10. Produces a verdict + findings + provenance.

## Failure modes during adoption

- **Audit interpretations in the verification frame.** "The auditor verifies that…" instead of "the auditor attempts to refute…". The grammatical inversion is Mechanism 3. Defaulting to verification grammar produces auditors that confirm rather than challenge.

- **Authority chain bottoming out in the audited role file.** "The auditor enforces the rule as written" re-introduces correlated blind spots. Authority must route to upstream governance for cases where the audited role file itself is the defect.

- **`audits:` frontmatter omitted.** Without it, the encompassment relationship is implicit rather than declared. Conformance criterion C-1 fails.

- **Reciprocal `audited_by:` pointer omitted.** Discoverability suffers; the relationship is only visible from one side. Conformance criterion C-2 fails.

- **Empty corpus omitted entirely.** Skipping the corpus when there are no entries to put in it. The directory tree must exist (with INDEX files) even when empty.

- **Loading the auditor role file into builder context.** The asymmetry is what produces independence. Loading the auditor file in the builder's session-start eliminates the mechanism. Conformance criterion C-14 fails.

- **Conflating builder rules with audit rules.** The audited role's rules live in the audited role's file. The auditor's rules live in the auditor's file. They are different rules. The auditor's Operating Rules govern *how to audit*; the audited role's Operating Rules govern *how to do the work*.

- **Skipping the scope-limits acknowledgment.** "The pattern doesn't solve X" is uncomfortable to write but essential.

## Migration from v0.1.x (same-file architecture)

Projects that adopted v0.1.x of this pattern have a `# §Audit-Variant` section in their audited role files. Migration to v0.2.0:

1. **Extract the §Audit-Variant section content** into a new `<audited-role-slug>-auditor.role.md` file structured per `template-auditor-role-file.md`. The per-criterion clauses become the auditor file's "Verification of audited role files" section; the cross-rule obligations become its "Cross-rule audit obligations" section; the encompassment / asymmetric-corpus / adversarial-default / re-derivation preamble distributes across the auditor file's Normative Requirements + Operating Rules.

2. **Remove the §Audit-Variant section** from the audited role file. Provenance update notes the migration.

3. **Add the reciprocal `audited_by:` pointer** to the audited role file's frontmatter.

4. **Bump versions** on both files (the audited role per project convention; the auditor at v0.1.0 as a new file).

5. **The asymmetric corpus** survives the migration unchanged in Categories A, B, C, E. Category D's contents relocate from the §Audit-Variant section to the new auditor file's body — same logical role, different location.

Adopters who prefer to stay on v0.1.x can pin to the `archive/v0.1.x-same-file-architecture` branch of this repository.

## Adoption checklist

A compact form for use during adoption:

- [ ] Step 1 — Audited role and verification structure identified.
- [ ] Step 2 — Auditor filename + display name chosen per convention.
- [ ] Step 3 — Upstream governance enumerated.
- [ ] Step 4 — Corpus location chosen; directory tree created; INDEX files seeded; access discipline stated in `audit-corpus/README.md`.
- [ ] Step 5 — Key-check categories enumerated for this role.
- [ ] Step 6 — Auditor role file authored:
  - [ ] Frontmatter: `audits:` populated; `follows_pattern:` populated.
  - [ ] Purpose + Scope.
  - [ ] Normative Requirements (adversarial default + derive-then-compare + citation + authority precedence + falsifiable-correctness + no-modification).
  - [ ] Operational Constraints (session-start load discipline + cold-context posture).
  - [ ] Inputs (audit subject + audited role file + upstream governance + asymmetric corpus + pattern reference).
  - [ ] Outputs (verdict + findings + provenance).
  - [ ] Verification of auditor's own work.
  - [ ] Operating Rules 1–7.
  - [ ] One audit-interpretation clause per audited-role check, refutation-framed.
  - [ ] Minimum three cross-rule obligations.
  - [ ] All six relational primitives.
  - [ ] Escalation & Halt Conditions.
  - [ ] Change Control + Provenance.
- [ ] Step 7 — Audited role file edited: `audited_by:` frontmatter pointer added; version bumped; provenance updated. Maintainers notified.

When the checklist is complete, the adoption is conformant with `auditor-pattern-spec.md` v0.2.0 at v0.1.0-empty corpus state. Conformance verification details live in `conformance-criteria.md`.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Major revision per architectural inversion from same-file (v0.1.x) to separate-file (v0.2.0).
- time: 2026-06-07
- summary: v0.2.0 — Rewritten seven-step adoption walkthrough for v0.2.0 separate-file architecture. Steps now create a new auditor role file rather than add a §Audit-Variant section to the audited role file. New step 7 adds the reciprocal `audited_by:` frontmatter pointer to the audited role file (the only required edit to that file under v0.2.0). What-changes-for-builder section simplified (builder behavior virtually unchanged; new frontmatter field is informational). What-changes-for-auditor section expanded (auditor is now a full role file with its own Operating Rules, Verification, Inputs). New "Migration from v0.1.x" section walks v0.1.x adopters through extracting their §Audit-Variant sections into separate auditor files. Failure modes updated with v0.2.0-specific cases (frontmatter pointer omissions, auditor-file-loaded-by-builder). Companion to auditor-pattern-spec.md v0.2.0 and template-auditor-role-file.md v0.2.0.
