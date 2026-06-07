---
name: adoption-guide
version: 0.1.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [guide, adoption, audit, governance]
references: auditor-pattern-spec.md v0.1.0, template-audit-variant-section.md v0.1.0, audit-corpus-spec.md v0.1.0
---

# Adoption Guide

This guide walks an adopter through instantiating the Project Prestidigitonium auditor pattern in an existing role file. It is the practical companion to `auditor-pattern-spec.md` (which states the pattern in principle) and `template-audit-variant-section.md` (which provides the copy-pasteable skeleton).

The guide assumes you have:

- A role file conforming to a base role template (Aire's `claude.role.base.md` or equivalent) with an explicit operating-rule or verification-check structure.
- An understanding of the role's domain sufficient to author per-rule audit clauses.
- A repository in which the role file and any associated governance specs live.

The guide does **not** assume you have a multi-role project, a sprint discipline, an existing audit history, or pre-existing failure-pattern documentation. The pattern adopts cleanly into a single-role project starting from scratch.

## Adoption in seven steps

### Step 1 — Confirm the role's rule structure

Identify what the audit interpretation clauses will attach to. Almost every Aire-shaped role has one of:

- A numbered **Operating Rules** section (e.g., Rules 1–13).
- A numbered **Verification** section (e.g., Checks V1–V12).
- A bulleted **Normative Requirements** section (each MUST as the unit).

The structure determines what `<rule-unit-name>` becomes in the §Audit-Variant heading. If the role uses multiple structures, choose the one most operationally consequential — usually Operating Rules when present, Verification when not.

If the role has none of these — for example, an exploratory role described only in prose — adopting the pattern requires first imposing rule structure. The pattern does not work on prose-only roles. Either author the rules before adopting, or defer adoption until the role matures.

### Step 2 — Choose the auditor's name

Per `auditor-pattern-spec.md` naming convention: `<Project> <Role> Auditor`.

Examples:
- A project called "Sketch" with an "Operator" role → **Sketch Operator Auditor**.
- A project called "Aire" with a "RoleSmith" role → **RoleSmith Auditor** (project name implicit; explicit form acceptable).
- A project called "Dispenser" with an "Arduino Engineer" role → **Dispenser Arduino Engineer Auditor**.

Avoid metaphor language. The name says what the auditor does; framing language belongs in design discussion, not the artifact.

### Step 3 — Identify upstream governance

Enumerate the specs that form the auditor's authority bedrock. These are the documents that say "what compliance means" independently of how the role file interprets it. Typical sources:

- The base role template (e.g., `claude/claude.role.base.md`).
- Kit specs the role references (briefing, verification, diagnostic, blocking, decision-logging, session-context, documentation, or your project's equivalents).
- The foundations or governance-floor spec for the project.
- Any project-level brief, contract, or acceptance criteria document.
- External standards or licenses the project commits to.

These get listed explicitly in the §Audit-Variant preamble. They are the answer to "if the role file is wrong, what is right?"

### Step 4 — Decide the asymmetric corpus location

Per `audit-corpus-spec.md`, the corpus lives in a directory tree. The default layout is `audit-corpus/` at the project root. Projects with multiple roles may either:

- **Share a single corpus**, organized internally by role and category.
- **Maintain per-role corpora**, e.g., `audit-corpus/sketch-operator/`, `audit-corpus/sketch-builder/`.

Per-role is recommended when domains diverge; shared is recommended when audit findings are likely to inform multiple roles.

At adoption time, create the directory tree with empty INDEX files and a brief `README.md` stating the access discipline. The corpus may be v0.1.0-empty — no entries — and the pattern still operates. Empty does not mean broken; it means new.

### Step 5 — Enumerate key-check categories for this role

Independent re-derivation (Mechanism 4) is role-specific. Determine which checks must be derive-then-compare and which may be read-then-verify. Common categories:

- **Re-measurement** — for roles producing numeric outputs (renders, simulations, measurements).
- **Re-classification** — for roles producing categorical outputs (decision classes, risk ratings, completion states).
- **Re-derivation of acceptance criteria** — for roles operating under written briefs whose criteria can be misread.
- **Rule-applicability mapping** — for roles operating in domains where which rules govern is itself contested.
- **Completeness check** — for roles producing artifacts where what's *missing* is harder to see than what's present.

The role file's §Audit-Variant preamble lists the categories. The per-rule clauses cite which categories apply where.

### Step 6 — Author the §Audit-Variant section

Copy the skeleton from `template-audit-variant-section.md` into the role file at the correct position (under `# Change Control` → `## Provenance`, as a top-level `# §Audit-Variant — …` heading).

Fill in:
- The heading: `<Project> <Role> Auditor's lens on the <N> <rule-unit-name>`.
- The encompassment preamble: upstream governance list (from Step 3).
- The asymmetric corpus access: paths (from Step 4) plus the discipline statement.
- The adversarial default: state it verbatim from the template.
- The independent re-derivation enumeration: key-check categories (from Step 5).
- The authority precedence: state it verbatim from the template.
- The scope limits acknowledgment: state it verbatim from the template.

Then for each rule in the role's rule structure (from Step 1), author one audit-interpretation clause:

- **What the auditor attempts to refute.** Phrase as a refutable proposition (the refutation frame is load-bearing).
- **Refutation mechanism.** Concrete enough that two independent auditors would converge on the same procedure.
- **Audit fail conditions.** When the refutation succeeds, or when the claim cannot be confirmed against the mechanism.

Then author the cross-rule obligations. The five recommended in the template (independent verification posture, scope ambiguity disclosure, authority chain check, stale rule-numbering refusal, memory-as-rule-source disclosure) are a reasonable starting set. Add project-specific obligations as needed.

### Step 7 — Bump version, update provenance, surface to maintainers

The role file just gained a substantial new section. Bump the role file's version per the project's versioning conventions (typically a minor bump: `0.x.y → 0.x+1.0` or `2.y.z → 2.y+1.0`, depending on whether your project treats audit-lens additions as minor or patch).

Update the role file's provenance with a summary noting:
- Addition of the §Audit-Variant section.
- Conformance with `auditor-pattern-spec.md` v<X.Y.Z>.
- The auditor's name (Step 2).
- The asymmetric corpus location (Step 4).

Surface the change to the role's maintainers and to whoever runs audit passes in your project. The pattern is operational the moment the section lands; it does not require additional infrastructure.

## What changes for the builder

Almost nothing. The builder reads the same role file it always read, with the same operating rules, the same governance compliance section, the same provenance. The §Audit-Variant section is present in the file but not on the builder's load list — its presence is structural, not behavioral.

The two real changes for the builder:

1. **The corpus exists.** The builder MUST NOT load `audit-corpus/` paths at session start. The role's session-context-load list (if explicitly enumerated) excludes corpus paths.
2. **Findings may surface.** When the auditor runs and identifies defects, the builder may receive routed findings (per the project's routing convention). These behave like any other governance flag; they do not require a new escalation channel.

## What changes for the auditor

Everything that wasn't already there. The auditor:

1. Loads the role file (including the §Audit-Variant section).
2. Loads the asymmetric corpus's active entries.
3. Loads upstream governance.
4. Loads project state (the brief, the contract, STATE.md if present).
5. Does NOT load the builder's auto-memory, session narrative, or any builder-specific context that is not part of the artifact under audit.
6. Applies the adversarial default to each refutation attempt.
7. Performs independent re-derivation on the enumerated key-check categories.
8. Cites corpus entries when applying their patterns; cites upstream governance when overriding the role file's rule; cites the user when deferring to a directive.

## Failure modes during adoption

The following adoption errors recur. Watch for them:

- **Audit clauses in the verification frame.** "The auditor verifies that…" instead of "the auditor attempts to refute…". The grammatical inversion is Mechanism 3 and is load-bearing. Defaulting to verification grammar produces auditors that confirm rather than challenge.

- **Authority chain bottoming out in the role file.** "The auditor enforces the rule as written" is a reading that re-introduces correlated blind spots. The authority chain must explicitly route to upstream governance for cases where the role file itself is the defect.

- **Empty corpus omitted entirely.** Adopters sometimes skip the corpus when there are no entries to put in it. The directory tree must exist (with INDEX files) even when empty. The discipline is to make growth structural rather than discretionary.

- **Loading the corpus into builder context "for transparency."** The asymmetry is what produces independence. Eliminating it eliminates the mechanism.

- **Treating §Audit-Variant as a checklist for the builder.** It is not. The builder reads execute-mode rules; the auditor reads audit interpretations. Conflating the two collapses the lens distinction.

- **Skipping the scope-limits acknowledgment.** "The pattern doesn't solve X" is uncomfortable to write but essential. Without it, downstream readers infer coverage the pattern doesn't actually provide.

## Adoption checklist

A compact form for use during adoption:

- [ ] Step 1 — Rule unit identified (`<rule-unit-name>` value chosen).
- [ ] Step 2 — Auditor named per convention.
- [ ] Step 3 — Upstream governance enumerated.
- [ ] Step 4 — Corpus location chosen; directory tree created; INDEX files seeded; access discipline stated in `audit-corpus/README.md`.
- [ ] Step 5 — Key-check categories enumerated for this role.
- [ ] Step 6 — §Audit-Variant section authored:
  - [ ] Heading per convention.
  - [ ] Encompassment preamble complete.
  - [ ] Asymmetric corpus access complete.
  - [ ] Adversarial default stated.
  - [ ] Independent re-derivation categories enumerated.
  - [ ] Authority precedence stated.
  - [ ] Scope-limits acknowledgment present.
  - [ ] One audit-interpretation clause per rule, refutation-framed.
  - [ ] Minimum three cross-rule obligations (five recommended).
- [ ] Step 7 — Role file version bumped; provenance updated; maintainers notified.

When the checklist is complete, the role is conformant with `auditor-pattern-spec.md` v0.1.0 at v0.1.0-empty corpus state. Conformance verification details live in `conformance-criteria.md`.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Initial draft.
- time: 2026-06-06
- summary: v0.1.0 — Initial adoption guide. Seven-step walkthrough from "role file with rules" to "conformant §Audit-Variant section landed" — rule-structure identification, auditor naming, upstream governance enumeration, corpus location decision, key-check enumeration, section authoring, version bump and provenance update. Includes what-changes-for-builder and what-changes-for-auditor sections, recurring adoption failure modes, and a compact checklist. Companion to auditor-pattern-spec.md and template-audit-variant-section.md.
