---
name: audit-corpus-spec
version: 0.2.6
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [governance, corpus, audit, asymmetric-knowledge]
references: auditor-pattern-spec.md
---

# Audit Corpus Specification

## Purpose

This specification defines the structure, growth rules, and access discipline for the **asymmetric corpus** — the body of materials that an auditor instance reads but the corresponding builder does NOT load by default. The corpus is the operational realization of Mechanism 2 in `auditor-pattern-spec.md`: it is what makes the auditor expert on how the role's reasoning fails, without exposing the builder to the salience bias of seeing its own failure modes catalogued.

## What lives in the corpus

The audit corpus contains five entry categories. A conformant implementation populates at least three of the five. (Empty categories are permitted at v0.1.0 of an adopting project — the corpus grows by use — but the categories MUST be declared and structurally present.)

### Category A — Prior audit findings

Durable record of defects caught in past audit passes. Each entry contains:
- A reference to the rule (or cross-rule obligation) the finding pertains to.
- The defect pattern (what the builder did, in terms that don't identify the specific session).
- The refutation mechanism that surfaced it (so future auditors can apply the same mechanism).
- The remediation that resolved it (so the same defect is not re-flagged when remediated).

Findings entries are append-only once written. Supersession (when a finding is reframed, merged with another, or determined to have been incorrect) is handled by adding a new entry that references the prior one, never by mutating history.

#### From finding to entry

Verdicts produced per `template-auditor-role-file.md`'s Outputs section carry a structured finding schema. Promotion of a finding to a Category A entry is a mechanical field mapping, not a reconstruction:

| Finding field | Category A entry destination |
|---|---|
| `id` | entry `id` (prefixed with the category letter: `A-<finding-id>`) |
| `check` | `applies_to_rules` |
| `corpus_refs` | candidate `informed` / `supersedes` linkage |
| `proposition` + `result` | the defect pattern (body) |
| `derived` + `observed` + `citation` | the refutation mechanism that surfaced it (body) |

The remediation field of the entry body is authored at promotion time (it is not present in the finding, because the finding predates the fix). Promotion is mechanical; the decision to promote remains human — not every finding warrants an entry, only those whose pattern future audit passes should detect by the same mechanism.

### Category B — Failure post-mortems

Narrative reconstructions of cases where a defect escaped audit and was caught downstream (by the user, by a later session, by production failure). Each entry contains:
- What was missed.
- The reasoning that produced the miss (in the auditor or in the builder, whichever applies).
- The change that would have caught it earlier — a new clause, a sharper refutation mechanism, an additional cross-rule obligation.
- A status field: `proposed`, `accepted`, `incorporated`. Once incorporated into the auditor's clauses, the post-mortem stays as historical context.

Post-mortems are the highest-friction entries because they require admission of an audit miss. They are also the highest-value entries because they document where the pattern needs sharpening.

### Category C — Drift catalogs

Documented patterns of how *this kind of builder* tends to fail under specific conditions. Each entry contains:
- The trigger condition (e.g., "high iteration count without measurable improvement", "first encounter with a new domain artifact type").
- The drift pattern (e.g., "optimism in interpretation labels", "tunnel vision on a single variable").
- The early-warning signal (what the auditor watches for *before* the drift produces a defect).
- The intervention (what the auditor does when the warning signal fires).

Drift catalogs are the closest analog to a "playbook" for the auditor. They are written in terms of the role's class (execution-heavy, authoring-heavy, research-heavy, integration-heavy), not the specific project.

The drift-pattern field SHOULD name the **causal mechanism**, not only the observed output. "Edits update the target line and never re-read its neighbors" explains *why* a class recurs; "references go stale" merely describes *that* it does. Patterns that name causes produce interventions; patterns that name outputs produce only vigilance.

### Category D — Self-referential auditor-file content

The auditor file's own bodies (Operating Rules, Verification of audited role files / per-criterion audit interpretations, Cross-rule audit obligations) ARE corpus material. They are loaded by the auditor at boot as the auditor's first-line behavioral specification. They are NOT loaded by the builder, because under the v0.2.0 separate-file architecture the auditor file lives at a different path (`<audited-role-slug>-auditor.role.md`) and the audited role's session-start load list does not reference it.

This is the lightest-friction asymmetric-corpus mechanism and is required by `auditor-pattern-spec.md` v0.2.0. Adopters cannot opt out of Category D — the auditor file's own content is corpus material by construction.

> v0.1.x note: under the same-file architecture, Category D was the §Audit-Variant section embedded in the audited role's file. v0.2.0's separate-file architecture relocates the same logical content to the auditor's own file. The category's role is unchanged; the location is.

### Category E — Project-specific asymmetric materials

Any additional materials a project determines should be auditor-only. Examples (illustrative, not prescriptive):
- A gotchas catalog with documented anti-patterns.
- A historical record of breaking changes and the audit signals that should have flagged them.
- An adversarial-input library when relevant to the role's domain.
- Cross-project audit findings, when an organization runs the pattern across multiple projects.

Category E is the open-ended category. Adopters extend it as their corpus matures.

## Structure on disk

The corpus is organized as a directory tree. The default layout (adopters may reorganize as long as the categories are addressable):

```
audit-corpus/
├── README.md                           — index + access discipline statement
├── A-findings/
│   ├── INDEX.md                        — by rule, by failure pattern, by date
│   ├── YYYY-MM-DD-<slug>.md            — one finding per file
│   └── …
├── B-post-mortems/
│   ├── INDEX.md
│   ├── YYYY-MM-DD-<slug>.md            — one post-mortem per file
│   └── …
├── C-drift-catalogs/
│   ├── INDEX.md
│   ├── <pattern-name>.md               — one pattern per file
│   └── …
├── D-self-referential/
│   └── README.md                       — explanatory pointer to the auditor role file (per Conformance Criterion C-15)
└── E-project-specific/
    ├── INDEX.md
    └── …
```

Category D's directory exists for symmetry and contains a `README.md` (per Conformance Criterion C-15) that explains the inclusion explicitly and points to the auditor role file (`<audited-role-slug>-auditor.role.md`) where the actual Category D materials live (Operating Rules, audit interpretations, cross-rule obligations). The directory does not contain category entries itself; the README is navigational, ensuring the directory survives version control and remains discoverable.

INDEX files are maintained per the project's existing index conventions (typically markdown tables, sorted by date or by rule). Indexes are NOT canonical — they are navigation aids — and may be regenerated from frontmatter.

## Entry frontmatter

All corpus entries (Categories A, B, C, E) carry frontmatter for indexability:

```yaml
---
id: <category-letter>-<sequence-or-slug>
category: <A | B | C | E>
status: <active | superseded | incorporated | retracted>
date: <YYYY-MM-DD of authoring>
applies_to_rules: [<rule-id>, …]              # for A and C; omit for B and E
applies_to_role: <role-id or "all">
supersedes: [<id>, …]                          # optional
superseded_by: [<id>, …]                       # optional
informed: [<id>, …]                            # optional — forward link to higher-order entries this informed (e.g., Category A → C promotion)
tags: [<tag>, …]
---
```

The `informed` field is the load-bearing forward-link for active-learning hooks (see §"Active-learning hooks" below). A Category A finding that contributed to a Category C drift catalog entry carries `informed: [C-<pattern-id>]`; programmatic analysis of `informed` linkages reveals which findings clustered into which patterns and where governance evolution traces back to specific evidence.

The `status` field is the load-time signal: an auditor loading the corpus reads `active` entries normatively, `superseded` and `retracted` entries as historical context (visible but not authoritative), `incorporated` entries as resolved (the post-mortem's lesson is now in the auditor role file's audit interpretations or cross-rule obligations; the entry stays for traceability).

## Growth rules

### How entries enter

- **Category A (findings)**: an entry is added when an audit pass surfaces a defect that future audit passes should be able to detect by the same mechanism. The author is the auditor that found the defect. The entry is added before the audit session closes; entries written from memory days later are explicitly lower-quality and should be flagged with a `recall_quality: post-hoc` tag.

- **Category B (post-mortems)**: an entry is added when a defect that escaped audit is caught downstream. The trigger is "what should have caught this earlier" — if the answer is non-trivial, it becomes a post-mortem. The author is whoever performs the post-mortem (auditor, user, builder, retrospectively).

- **Category C (drift catalogs)**: an entry is added when a pattern of failure (not a single instance) becomes legible. This typically requires three or more Category A findings of the same shape before the pattern is named. Cataloging too eagerly produces noise; cataloging too late means future auditors waste cycles re-discovering the pattern.

- **Category E (project-specific)**: governed by the project's own conventions, but entries MUST follow the standard frontmatter for consistency.

### How entries leave

Entries do not leave. They are marked superseded or retracted but remain in place for traceability. The corpus is append-only at the file level.

Supersession is the normal lifecycle: a finding from 2026-04 may be superseded by a sharper finding from 2026-08, with the older entry's `status` changed to `superseded` and `superseded_by` populated.

Retraction is the failure lifecycle: an entry determined to be incorrect (the "defect" was not a defect; the pattern named was not real) is marked `retracted` with a brief explanation in the body. Retracted entries stay because the auditor's reasoning at the time of the retraction is itself useful failure-pattern material.

### When entries get incorporated

Post-mortem entries with `status: accepted` should drive concrete changes to the auditor role file's audit interpretations or cross-rule obligations. When the change lands, the post-mortem's status flips to `incorporated`. This is the corpus's feedback loop into the auditor role file.

Findings entries can drive Drift catalog entries (Category A → Category C). When three or more findings name the same pattern, the auditor (or a maintainer) authors a Category C entry capturing the pattern, and the contributing Category A entries link forward via `informed: [C-pattern-id]`.

## Access discipline

The corpus's value depends on its asymmetry. The discipline that maintains asymmetry:

1. **Builder MUST NOT load the corpus by default.** The audited role's session-start load list does not include any `audit-corpus/` paths AND does not reference the auditor role file (`<audited-role-slug>-auditor.role.md`). This is the primary discipline. Under v0.2.0's separate-file architecture, the prohibition extends to the auditor file itself, because that file's body is Category D corpus material.

2. **Auditor MUST load the corpus by default.** The auditor role file's session-start load list includes the audited role file (per `audits:` frontmatter, for domain fluency), upstream governance, and the active entries in all populated corpus categories. Superseded and retracted entries are accessible on demand but not loaded eagerly.

3. **Loading is asymmetric but readable.** The corpus is not hidden — it lives in the project's repository, version-controlled, visible to anyone reading the project. The asymmetry is operational (who loads what at session start), not access-controlled.

4. **The builder may consult the corpus on demand**, but only when the auditor explicitly cites a corpus entry in a verdict, and only the specific entry cited. This handles the case where a builder needs to understand why a finding was raised. Bulk-loading the corpus into the builder's working context is non-conformant.

5. **Cross-corpus pollination is conformant.** Auditors of related roles MAY load each other's corpora when domain overlap is substantive (e.g., two execution roles in the same project may share Category C drift catalogs). The discipline that prevents builders from loading auditor corpora applies regardless of which corpus is being considered.

## Bootstrapping

A new adopting project will have an empty corpus. The pattern works at v0.1.0-empty — the four mechanisms are operational, just with thin asymmetric material — and grows from there. Bootstrap recommendations:

- **Seed Category C with the role's class-level drift patterns** if the role's class is one that has documented patterns elsewhere (e.g., execution-heavy roles inherit "optimism in interpretation" as a seed pattern; authoring-heavy roles inherit "silent requirement drop" as a seed pattern).
- **Treat the first three audit sessions as Category A populating sessions.** The yield will be modest; that's expected.
- **Run a post-mortem after the first downstream defect.** Even if it's small. The post-mortem's structural quality matters more than the defect's size, because subsequent post-mortems will compound on the precedent set.

The corpus matures in months, not sessions. v0.1.0-empty is fine.

## Anti-patterns

The following corpus practices undermine the pattern:

- **Mutating entries** instead of superseding them. Loses the trace of the auditor's reasoning evolution.
- **Loading the corpus into builder context** "for transparency". The salience bias the asymmetry exists to prevent re-emerges immediately.
- **Treating Category C as predictions.** Drift catalogs document patterns observed in this role's class; they are not forecasts. An auditor that asserts "this builder will exhibit pattern X" before the trigger fires is over-applying.
- **Cataloging single-instance findings as Category C patterns.** Wait for three.
- **Writing post-mortems without the "what would have caught this" field.** Without it, the post-mortem is a confession; with it, the post-mortem is corpus material.

## Active-learning hooks (forward-compatible)

The corpus is structured such that future tooling can analyze it programmatically without schema changes. This section documents the hooks that enable that analysis, so adopters authoring entries today produce material that downstream active-learning extensions can use tomorrow.

Active-learning extensions themselves are out of scope for this specification. They are anticipated in a future companion specification (working title: `active-learning-spec.md`) authored when accumulated real corpora provide enough evidence to design against. v0.1.x deliberately defers the design rather than building against an empty corpus.

### Machine-readable surfaces

The following frontmatter fields are designed to be both human-readable and machine-parsable. Adopters SHOULD populate them consistently:

- **`tags`** — controlled vocabulary at the project's discretion; the primary clustering signal. Tagging discipline matters: ad-hoc tags produce noise; consistent tags enable pattern detection.
- **`applies_to_rules`** — the rule-level grouping signal. Findings clustered on the same rule are the most direct evidence that the rule may be ambiguously written or that the audit clause needs sharpening.
- **`applies_to_role`** — the role-level grouping signal. Patterns observed across multiple roles within the same role-class (execution-heavy, authoring-heavy) become class-level drift catalogs.
- **`status`** — the lifecycle signal. `active` entries inform present audits; `superseded` and `retracted` entries inform corpus health analysis (high retraction rates suggest auditor overreach; high supersession rates suggest pattern maturation).
- **`supersedes` / `superseded_by`** — the lineage signal. Tracing supersession chains reveals how the auditor's understanding has evolved.
- **`informed`** — the cross-category linkage signal. Category A findings citing the Category C pattern they contributed to enable provenance tracing from governance evolution back to specific evidence.

### Detection-layer signals (future Layer 1)

Future detection tooling will likely surface flags including (illustrative, not prescriptive):

- **Clustering threshold** — N+ Category A findings sharing a tag or `applies_to_rules` value → "consider authoring a Category C drift pattern."
- **Pattern decay** — Category C entries with no recent `informed` references from new findings → "pattern may be stale; review for retirement."
- **Rule ambiguity** — finding cluster on a specific rule → "the rule itself may be ambiguously written; consider revision."
- **Role-file defect signal** — auditor citing the same upstream-governance violation repeatedly → "this is a role-file defect, not an instance defect."
- **Incorporation lag** — Category B post-mortems marked `accepted` for extended periods without `incorporated` status → "lesson identified but not landed; consider scheduling clause update."

Detection-layer signals do not modify state. They surface observations for human action. Their value is removing the cognitive overhead of pattern-spotting in growing corpora.

### Promotion-layer hooks (future Layer 2)

Promotion-layer tooling drafts candidate updates for human review and acceptance. The corpus structure supports this without schema changes — drafts inherit frontmatter from the contributing entries; humans review the proposed material against the principles in `auditor-pattern-spec.md` and accept, edit, or reject. The corpus's append-only discipline ensures that promotion-layer drafts are visible artifacts (committed as candidates with `status: proposed`), not hidden state mutations.

### Evolution-layer caution (future Layer 3)

Self-modifying governance — the auditor editing its own clauses based on accumulated evidence — is a category of change this pattern is deeply skeptical of. The auditor pattern exists to catch builders that drift toward easier-to-pass interpretations of their own rules; the same skepticism applies recursively to auditors that might drift toward easier-to-pass interpretations of *their* own clauses.

Adopters considering evolution-layer extensions SHOULD ensure:

- Every state change is a versioned git commit with provenance.
- Conformance criteria scan flags clause weakening as a defect class.
- Human review remains the default path, even for routine incorporations.
- The pattern's `auditor-pattern-spec.md` § "What this pattern does NOT solve" is updated to reflect any new failure modes the evolution-layer introduces.

Evolution-layer extensions are not anticipated for v0.x of this specification. They may never be appropriate. Future maintainers: apply skepticism.

### Discipline for adopters today

To produce active-learning-ready corpora without speculating on extension design:

1. **Populate frontmatter completely.** Optional fields are optional now; they may become load-bearing later. Populate them when the information exists.
2. **Maintain tag vocabularies.** Author a short `tag-vocabulary.md` in `audit-corpus/` listing canonical tags and their meanings. Refactor tags via supersession (new entry with corrected tags references the old; old marked `superseded`), never by editing history.
3. **Backfill `informed` when promotions happen.** When a Category A → Category C promotion occurs, edit the contributing Category A entries' `informed` fields. This is one of the few permitted edits to existing entries — it is metadata maintenance, not history mutation.
4. **Don't speculate.** Do not pre-author entries for patterns that haven't been observed. Active-learning hooks reward real evidence, not anticipated evidence.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Initial draft.
- time: 2026-06-07
- summary: v0.1.0 — Initial specification of the audit corpus. Defines five entry categories (prior findings, failure post-mortems, drift catalogs, self-referential §Audit-Variant clauses, project-specific materials), default disk layout, entry frontmatter schema, growth rules (entry, supersession, incorporation), access discipline (asymmetric load, visible but not eager-loaded by builder), bootstrapping guidance for v0.1.0-empty corpora, and anti-pattern catalog. Operationalizes Mechanism 2 of auditor-pattern-spec.md v0.1.0. v0.1.1 (2026-06-06) — Adds the `informed` field to the entry frontmatter schema (forward-link from contributing findings to higher-order entries they informed) and a new §"Active-learning hooks" section documenting the machine-readable surfaces, detection-layer signals, promotion-layer hooks, evolution-layer caution, and adopter discipline that produce active-learning-ready corpora today without committing to extension design. v0.2.0 (2026-06-07) — Relocates Category D's content from the §Audit-Variant section (same-file pattern, v0.1.x) to the auditor role file's own body (separate-file pattern, v0.2.0) to align with `auditor-pattern-spec.md` v0.2.0. Access discipline updated: the "builder MUST NOT load corpus" prohibition extends to the auditor role file itself under v0.2.0, because that file's body is Category D corpus material. Category role unchanged; location is. Other categories (A, B, C, E), entry frontmatter schema, growth rules, and active-learning hooks unchanged from v0.1.1. v0.2.2 (2026-06-07) — Per Sketch Main Auditor's v0.2.0 review: (E2) corrects the frontmatter `references:` field that was still pinning `auditor-pattern-spec.md v0.1.0` after the v0.2.0 body update. (E3) scrubs three stale §Audit-Variant references that survived the v0.2.0 pass: the disk-layout diagram at the Category D entry (was "no files — Category D lives in the role file's §Audit-Variant section"; now correctly shows `README.md` present); the status-field explanation at growth rules (was "the post-mortem's lesson is now in the §Audit-Variant clauses"; now correctly references the auditor file's audit interpretations / cross-rule obligations); the post-mortem incorporation rule (same fix). (E4) reconciles the prior internal contradiction between this spec and `conformance-criteria.md` C-15 — both now agree that Category D's directory contains a `README.md` pointing to the auditor file (the empty-directory alternative didn't survive git anyway, and the README is the navigational artifact that makes the inclusion discoverable). No structural or normative changes; v0.2.0 implementations remain conformant after applying the same internal scrub to their own corpora. v0.2.3 (2026-06-11) — Per ASA fresh-pass review: adds §"From finding to entry" under Category A, defining the mechanical field mapping from the structured finding schema (introduced in `template-auditor-role-file.md` v0.2.3 Outputs) onto Category A entry frontmatter and body. Closes the previously undefined gap between "auditor MAY draft a candidate Category A entry" (template Operating Rule 5) and the entry schema this spec defines — promotion is now a field mapping rather than a reconstruction, which also gives future active-learning Layer 1 tooling a parseable verdict-to-corpus trail. Promotion remains a human decision; only the mechanics are specified. v0.2.5 (2026-06-11) — Category C guidance gains the cause-over-output authoring rule: drift-pattern fields SHOULD name the causal mechanism, not only the observed output. Sourced from a live instance: a version pin survived three revisions because it sat one line below the edits ("proximity without participation") — the causal statement produced an intervention (pin-free phrasing) where the output statement had produced only repeated corrections. v0.2.6 (2026-06-11) — References field converted to pin-free per the same intervention this spec now teaches; version aligned to the release lockstep. No content changes.
