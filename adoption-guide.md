---
name: adoption-guide
version: 0.3.3
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [guide, adoption, audit, governance]
references: auditor-pattern-spec.md, template-auditor-role-file.md, audit-corpus-spec.md
---

# Adoption Guide

This guide walks an adopter through instantiating the Project Prestidigitonium auditor pattern in an existing role-based AI governance system under v0.2.0's separate-file architecture. The adopter creates a new auditor role file alongside the existing audited role file; the audited role file gets a single frontmatter pointer added but is otherwise unchanged.

This is the practical companion to `auditor-pattern-spec.md` (which states the pattern in principle), `template-auditor-role-file.md` (which provides the copy-pasteable skeleton), and `audit-corpus-spec.md` (which defines the asymmetric corpus).

The guide assumes you have:

- A role file conforming to a base role template (Aire's `claude.role.base.md` or equivalent).
- An understanding of the role's domain sufficient to author per-criterion audit interpretations.
- A repository in which the role file and associated governance specs live.

The guide does **not** assume a multi-role project, sprint discipline, existing audit history, or pre-existing failure-pattern documentation. The pattern adopts cleanly into a single-role project from scratch.

## Adoption in ten steps

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
- `audits_version: <the audited role file's current version>` — the version the per-criterion interpretations are authored against (Criterion C-21, Recommended)
- `follows_pattern: Project Prestidigitonium v0.5.0`
- `audit_posture: <artifact-verdict-only | continuous-monitoring | hybrid>`
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

### Step 7 — Add reciprocal pointer to the audited role file, bump versions

Edit the audited role's frontmatter to add: `audited_by: <audited-role-slug>-auditor.role.md`. This is the only required edit to the audited role file under v0.2.0. (Optional: scrub any deprecated language, update governance references — but those are independent housekeeping, not adoption requirements.)

Bump the audited role's version per the project's versioning conventions (minor bump typical: it gained a frontmatter pointer and an audit relationship).

Update both files' Provenance subsections with summaries noting:
- The new auditor file's existence.
- Reciprocal pointer added to the audited role.
- Conformance with `auditor-pattern-spec.md` v0.2.0+.
- The asymmetric corpus location.

### Step 8 — Wire project-harness routing so auditor instances actually bind to the auditor file

This step exists because the previous seven could all be completed correctly — every file conformant, every pointer reciprocal, every criterion passed — and the adoption could still sit inert. Project harnesses (CLAUDE.md anchor protocols, role-invocation conventions, IDE integration shims) typically hardcode which role file gets loaded at session start. If that hardcoded path always loads the audited role's file, then auditor invocations will silently anchor to the builder's content, never load the auditor's own role file, and produce no audits — while looking fully conformant on inspection.

None of the file-level conformance criteria catch this failure, because the criteria evaluate file-level conformance, not project-harness routing. Routing is project-environment configuration, not a pattern-level concern; but the obligation to update it is part of adoption.

**Update the project's session-start / anchor routing** so that auditor invocations load `<audited-role-slug>-auditor.role.md` rather than the audited role's file. Use one of the three strategies documented in §"Project CLAUDE.md routing under separate-file architecture" below:

- **Strategy A (recommended)** — separate CLAUDE.md per role/auditor directory.
- **Strategy B** — single CLAUDE.md with explicit instance dispatch.
- **Strategy C** — auditor invocation bypasses CLAUDE.md auto-load.

Document the chosen strategy in the auditor file's Provenance and in the project's CLAUDE.md(s) so downstream readers understand the bootstrap topology.

**Smoke test**: invoke the auditor; confirm the auditor's role file (not the audited role's) was the session anchor. If the auditor instance loads the audited role file at session start, Step 8 is incomplete regardless of how clean Steps 1–7 are. This is the most common silent adoption failure under v0.2.0 and is invisible to conformance review of the files alone.

**Model-diversity note (same invocation-environment territory as routing).** When wiring how auditor invocations launch, also decide *what model* they launch on. Recommendation: run the auditor on a **different model** than the builder, **at least as capable** as the builder's (never weaker — a weaker auditor manufactures capability blind spots), with an **elevated thinking tier** when available. **Express the pairing as a self-healing standing rule, not a brittle dated name:** state your preferred pairing and use it whenever accessible; when a preferred model is unavailable (it can be withdrawn earlier than its announced window), **auto-defer to the most current leading model available at the most appropriate settings** rather than carrying a stale name. When only one model is available, hold decorrelation by the separate-context / separate-file / asymmetric-corpus separation plus an **effort/method asymmetry** — the auditor at a higher effort/method tier than the builder (elevated thinking tier; multi-agent adversarial orchestration where available), never by downgrading the builder; same-model-higher-tier alone improves thoroughness within the same lens but does not by itself decorrelate the shared priors that file separation cannot remove. Record the rule, dated, in the auditor file's Operational Constraints, and re-pin only when the model or effort/method assignment materially changes. Full rationale in the README §"Deployment recommendation: model diversity."

### Step 9 — Wire the comms channel (bootstrap pointer now; the auditor does the rest at first boot)

Per `comms-spec.md`, builder↔auditor mail is a message-per-file inbox system that the **auditor** stands up at its first boot: it creates the `.comms/` tree if absent and files the activation directive into the builder's inbox — the channel's first message is the builder's operating manual. The adopter's part of Step 9 is two small things:

1. **Confirm the auditor file carries the comms duties** (the template skeleton includes them: read-then-bring-up discipline — `heartbeat up <slug>` where the reliability provisions are adopted, with `heartbeat wait` activation and the `.done` reader-cursor — first-boot setup including `cursors/`, `heartbeats/`, `.comms/bin/`, and `comms.conf`, and self-triggering housekeeping).
2. **Add the bootstrap pointer to the builder's session-entry configuration** (the CLAUDE.md you touched in Step 8, or equivalent): one line directing the builder to read `.comms/inbox-<builder-slug>/` newest-first at session start and bring its watch up (`heartbeat up <slug>`). Without this line, the activation directive waits in an inbox nothing opens — the directive cannot bootstrap itself.

The audited role file is NOT edited for comms — Step 7's single-required-edit property holds. Embedding the comms discipline in the audited role's Operational Constraints is optional hardening at the adopter's discretion.

### Step 10 — Surface

Notify maintainers and audit-pass operators. The pattern is operational the moment Steps 1–9 are complete and the corpus tree exists; the comms channel completes itself at the auditor's first boot.

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

- **Loading the auditor role file into builder context.** The asymmetry is what produces independence. Loading the auditor file in the builder's session-start eliminates the mechanism. Conformance criterion C-16 fails.

- **Conflating builder rules with audit rules.** The audited role's rules live in the audited role's file. The auditor's rules live in the auditor's file. They are different rules. The auditor's Operating Rules govern *how to audit*; the audited role's Operating Rules govern *how to do the work*.

- **Skipping the scope-limits acknowledgment.** "The pattern doesn't solve X" is uncomfortable to write but essential.

## Project CLAUDE.md routing under separate-file architecture

Most Aire-shaped projects use a `CLAUDE.md` file at the project's `claude/` directory as the entry-point bootstrap: it tells the runtime which role file to read and lists governance imports. Under v0.1.x's same-file architecture, this was a non-issue — both lenses lived in the same file, so any CLAUDE.md routing to that file routed correctly. Under v0.2.0's separate-file architecture, the routing matters: the CLAUDE.md that anchors the builder MUST NOT route the auditor to the audited role's content, because doing so contaminates the auditor's cold-context posture before its own role file ever loads.

This is a project-environment concern, not a property of the pattern itself, but adopters routinely hit it during onboarding. Three viable routing strategies:

### Strategy A — Separate CLAUDE.md per role/auditor directory (recommended)

Mirror the file-boundary asymmetry at the directory level. Most adopters should choose this.

```
project/
├── claude/                                  # builder's environment
│   ├── CLAUDE.md                            # → "Read role file: <role>.role.md"
│   ├── <role>.role.md                       # audited role file
│   └── (governance specs, kits, etc.)
├── audit/                                   # auditor's environment
│   ├── CLAUDE.md                            # → "Read role file: <role>-auditor.role.md;
│   │                                            audited role lives at ../claude/<role>.role.md"
│   ├── <role>-auditor.role.md               # auditor role file
│   └── audit-corpus/                        # asymmetric corpus
└── ...
```

**Invocation pattern.** Builder invocations happen in `claude/` (the existing pattern). Auditor invocations happen in `audit/` (new). The runtime reads the CLAUDE.md present in the directory it was invoked from — natural directory-determined dispatch with no conditional logic required.

**Pros:**
- File-boundary asymmetry extends to bootstrap-boundary asymmetry. The builder's CLAUDE.md never references the auditor file's existence; salience contamination is impossible by construction.
- No "which instance am I" runtime ambiguity; the working directory answers the question.
- Asymmetric corpus has a natural home (`audit/audit-corpus/`) that's accessible to the auditor by relative path and structurally invisible to the builder's bootstrap.
- Matches the cleanest interpretation of v0.2.0's underlying principle: enforcement by structural separation rather than by reminder.

**Cons:**
- Two CLAUDE.md files instead of one; users need to know which directory to invoke from.
- Cross-directory references (`audit/CLAUDE.md` pointing to `../claude/<role>.role.md` as the audited role) introduce a path coupling that has to be maintained if directories are reorganized.

### Strategy B — Single CLAUDE.md with explicit instance dispatch

Acceptable when invocation context is naturally explicit (e.g., the user always announces "acting as builder" or "acting as auditor" at session start).

```markdown
# Project CLAUDE.md

## Instance dispatch
- If invoked as **<Role>** (build mode): read `<role>.role.md`. Do NOT read `<role>-auditor.role.md` or `audit-corpus/`.
- If invoked as **<Role> Auditor** (audit mode): read `<role>-auditor.role.md` (which loads `<role>.role.md` as a declared input). Asymmetric corpus at `audit-corpus/`.

## Governance (common)
...
```

**Pros:**
- Single bootstrap file; no directory reorganization.
- Lower onboarding friction for projects already organized around a single `claude/` directory.

**Cons:**
- Asymmetry depends on the runtime correctly dispatching from invocation context — which is exactly the load-discipline failure class v0.2.0 was meant to escape. Builder invocations that ignore the dispatch language re-introduce the v0.1.x failure mode.
- The single CLAUDE.md *mentions* the auditor file even in the builder's path — salience contamination risk, though smaller than v0.1.x's full-text contamination.
- "Which instance am I" is decided per-invocation by language convention rather than by directory structure.

Use Strategy B only when Strategy A's directory cost is prohibitive or when dispatch is explicit and disciplined.

### Strategy C — Auditor invocation bypasses CLAUDE.md auto-load

In environments where the runtime supports direct invocation of a role file without reading CLAUDE.md (e.g., explicit `--role <path>` invocation flags, or non-Claude-Code platforms with different bootstrap models), the auditor can be invoked directly against its role file, skipping CLAUDE.md entirely.

**Pros:**
- Simplest architectural model — only the builder reads CLAUDE.md; auditor is "out of band."
- Zero CLAUDE.md changes required.

**Cons:**
- Only viable when the runtime supports it.
- Audit invocations are no longer first-class members of the project's bootstrap ecosystem; documenting them as "different from how the builder is invoked" creates onboarding friction.

Use Strategy C only when runtime constraints favor it.

### Recommendation

For new adoptions: **Strategy A.** Mirror the file-boundary asymmetry at the directory level. The two-directory cost is modest; the structural cleanliness is substantial.

For migrations from v0.1.x: **Strategy A** if directory reorganization is acceptable; **Strategy B** if not. Strategy B is a defensible interim during transition; recommend Strategy A as the stable end state.

For all adoptions: **document the chosen strategy** in the auditor's role file Provenance and in the project's CLAUDE.md so downstream readers understand the bootstrap topology.

### Conformance note

The pattern spec (`auditor-pattern-spec.md`) does not mandate a routing strategy. Project CLAUDE.md routing is project-environment configuration, not a pattern-level concern. However, all three strategies satisfy the underlying requirement: the builder's session-start path MUST NOT load the auditor role file or the asymmetric corpus. Strategies that fail that requirement are non-conformant with `conformance-criteria.md` Criterion C-16.

## Migration from v0.1.x (same-file architecture)

Projects that adopted v0.1.x of this pattern have a `# §Audit-Variant` section in their audited role files. Migration to v0.2.0:

1. **Extract the §Audit-Variant section content** into a new `<audited-role-slug>-auditor.role.md` file structured per `template-auditor-role-file.md`. The per-criterion clauses become the auditor file's "Verification of audited role files" section; the cross-rule obligations become its "Cross-rule audit obligations" section; the encompassment / asymmetric-corpus / adversarial-default / re-derivation preamble distributes across the auditor file's Normative Requirements + Operating Rules.

2. **Remove the §Audit-Variant section** from the audited role file. Provenance update notes the migration.

3. **Add the reciprocal `audited_by:` pointer** to the audited role file's frontmatter.

4. **Bump versions** on both files (the audited role per project convention; the auditor at v0.1.0 as a new file).

5. **The asymmetric corpus** survives the migration unchanged in Categories A, B, C, E. Category D's contents relocate from the §Audit-Variant section to the new auditor file's body — same logical role, different location.

The same-file architecture is retired as of v0.2.6; there is no supported pin point for it. This migration section is retained as the supported path for deployments that still carry `# §Audit-Variant` sections — the extraction source is the deployment's own role files, not this repository.

## Adoption checklist

A compact form for use during adoption:

- [ ] Step 1 — Audited role and verification structure identified.
- [ ] Step 2 — Auditor filename + display name chosen per convention.
- [ ] Step 3 — Upstream governance enumerated.
- [ ] Step 4 — Corpus location chosen; directory tree created; INDEX files seeded; access discipline stated in `audit-corpus/README.md`.
- [ ] Step 5 — Key-check categories enumerated for this role.
- [ ] Step 6 — Auditor role file authored:
  - [ ] Frontmatter: `audits:` populated; `audits_version:` pinned to the audited role's current version; `follows_pattern:` populated; `audit_posture:` declared.
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
- [ ] Step 7 — Audited role file edited: `audited_by:` frontmatter pointer added; version bumped; provenance updated.
- [ ] Step 8 — Project-harness routing wired:
  - [ ] Routing strategy chosen (A / B / C from §"Project CLAUDE.md routing").
  - [ ] Strategy documented in auditor file's Provenance + project CLAUDE.md.
  - [ ] Smoke test passed: auditor invocation confirmed to load the auditor's role file at session start, NOT the audited role's file.
  - [ ] Builder/auditor model/method pairing decided (auditor ≥ builder; different model preferred; **self-healing rule** — use the preferred pairing if accessible, else auto-defer to the most current leading model; effort/method asymmetry when only one model is available) and recorded, dated, in the auditor file's Operational Constraints.
- [ ] Step 9 — Comms channel wired:
  - [ ] Auditor file carries the comms duties (read-then-arm, first-boot setup, housekeeping).
  - [ ] Bootstrap pointer added to the builder's session-entry configuration (read inbox newest-first at session start + arm watch).
- [ ] Step 10 — Maintainers + audit-pass operators notified.

When the checklist is complete, the adoption is conformant with `auditor-pattern-spec.md` v0.2.0+ at v0.1.0-empty corpus state. Conformance verification details live in `conformance-criteria.md`.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Major revision per architectural inversion from same-file (v0.1.x) to separate-file (v0.2.0).
- time: 2026-06-07
- summary: v0.2.0 — Rewritten seven-step adoption walkthrough for v0.2.0 separate-file architecture. Steps now create a new auditor role file rather than add a §Audit-Variant section to the audited role file. New step 7 adds the reciprocal `audited_by:` frontmatter pointer to the audited role file (the only required edit to that file under v0.2.0). What-changes-for-builder section simplified (builder behavior virtually unchanged; new frontmatter field is informational). What-changes-for-auditor section expanded (auditor is now a full role file with its own Operating Rules, Verification, Inputs). New "Migration from v0.1.x" section walks v0.1.x adopters through extracting their §Audit-Variant sections into separate auditor files. Failure modes updated with v0.2.0-specific cases (frontmatter pointer omissions, auditor-file-loaded-by-builder). Companion to auditor-pattern-spec.md v0.2.0 and template-auditor-role-file.md v0.2.0. v0.2.1 (2026-06-07) — Adds new "Project CLAUDE.md routing under separate-file architecture" section per Sketch Main Auditor's observation that hardcoded CLAUDE.md references to the audited role file's content contaminate the auditor's cold-context posture at boot under v0.2.0. Three routing strategies documented: Strategy A — separate CLAUDE.md per role/auditor directory (recommended; mirrors file-boundary asymmetry at directory level); Strategy B — single CLAUDE.md with explicit instance dispatch (acceptable when invocation context is naturally explicit); Strategy C — auditor invocation bypasses CLAUDE.md auto-load (only viable when runtime supports it). All three satisfy Criterion C-16 when implemented correctly; the pattern spec does not mandate a strategy. Recommendation: Strategy A for new adoptions; Strategy A for migrations where feasible (B as defensible interim). v0.2.2 (2026-06-07) — Per Sketch Main Auditor's v0.2.0 review: (E1) corrects the wrong-criterion reference in the failure-modes list (was C-14, should be C-16 — the load-list discipline criterion). Adds new Step 8 "Wire project-harness routing so auditor instances actually bind to the auditor file" addressing the gap the auditor flagged as Q1 — a fully conformant adoption could sit inert if the project harness keeps routing auditor invocations to the audited role's file; none of the nineteen criteria catch this failure because routing is project-environment configuration. Step 8 names the obligation, references the three routing strategies, and requires a smoke test (invoke the auditor; confirm correct session anchor). Old "Surface" content split out as Step 9. Adoption checklist extended with Step 8 (strategy chosen + documented + smoke-test passed) and Step 9. Total step count: nine. v0.2.3 (2026-06-11) — Per ASA fresh-pass review: corrects the frontmatter `references:` field, which still pinned all three sibling specs at v0.2.0 (the E2 defect class recurring in this file after v0.2.2 fixed it in `audit-corpus-spec.md`). Replaces Step 8's criterion count ("none of the nineteen conformance criteria") with count-free phrasing — the count was already stale at landing because Step 8 and Criterion C-20 shipped in the same release, and embedded counts are the drift class this same release documented. Step 6 frontmatter list extended with `audits_version:` (Criterion C-21, Recommended) and `audit_posture:`; `follows_pattern:` example updated. Adoption checklist's Step 6 frontmatter line extended to match. v0.2.4 (2026-06-11) — Adds model-diversity note to Step 8 (model selection is invocation-environment configuration, same territory as routing): auditor runs on a different model than the builder, at least as capable (capability floor — never weaker), with elevated thinking tier when available; same-model-higher-tier alone is a fallback that improves thoroughness but does not decorrelate shared priors. Checklist Step 8 extended with the model-pairing decision item. Full rationale lives in README §"Deployment recommendation: model diversity"; adopters pin dated specifics in their auditor file's Operational Constraints. v0.2.5 (2026-06-11) — References-field refresh only (`audit-corpus-spec.md` → v0.2.5); no body changes. v0.2.6 (2026-06-11) — Same-file retirement: the migration section's closing line no longer offers the archive branch as a pin point (branch removed; section retained as the supported path for remaining same-file deployments, whose extraction source is their own role files). References field converted to pin-free per the drift catalog's intervention — filenames only; versions live in each file and the CHANGELOG, ending the references-refresh treadmill this file exemplified twice (R2a, v0.2.5). v0.3.0 (2026-06-11) — New Step 9 (comms wiring): adopter confirms the auditor file's comms duties and adds the one-line bootstrap pointer to the builder's session-entry configuration; the auditor's first boot creates the `.comms/` tree and files the activation directive. Surface renumbered to Step 10; header and checklist updated to ten steps; Step 6 `follows_pattern:` example advanced to v0.3.0. The audited role file remains untouched by comms — Step 7's single-required-edit property holds. v0.3.1 (2026-06-11) — Step 9 duty enumeration aligned to the comms-spec v0.4.0 watch-economy provisions (watch-lifetime rule, cursors and commit-signal hooks in setup, hook presence check in housekeeping) and made version-free (the "v0.3.0 template skeleton" phrasing was a prose pin of the C-stale class). No step structure changes. v0.3.2 (2026-06-15) — Step 9 comms-duty enumeration synced to `comms-spec.md` v0.5.0 §Reliability: read-then-bring-up (`heartbeat up <slug>`) with `heartbeat wait` activation and the `.done` reader-cursor; first-boot setup now lists `cursors/`, `heartbeats/`, `.comms/bin/`, and `comms.conf`; commit-signal hook installation and the hook presence check removed (superseded). Step 6 `follows_pattern:` example advanced v0.3.0 → v0.5.0 (comms duties moved — a conformed surface). No step structure changes. v0.3.3 (2026-06-15) — Model-diversity note (Step 8) + checklist upgraded to the **self-healing model/method-pairing rule**: prior guidance recorded a dated hard-pin, which went stale in a running instance when a preferred model was withdrawn before its announced window; the note now reads prefer-the-preferred-pairing-if-accessible, else auto-defer to the most current leading model at appropriate settings, and adds the effort/method asymmetry (auditor at a higher effort/method tier than the builder, never by downgrading the builder) as the same-model decorrelation mechanism. Re-pin only on a material model/method change. Companion to README §model-diversity and `template-auditor-role-file.md` v0.3.6. No step structure or criteria changes.
