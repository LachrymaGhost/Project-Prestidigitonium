---
name: audit-corpus-spec
version: 0.1.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [governance, corpus, audit, asymmetric-knowledge]
references: auditor-pattern-spec.md v0.1.0
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

### Category D — Self-referential §Audit-Variant clauses

The audit-interpretation clauses authored in the role file's §Audit-Variant section ARE corpus material. They are loaded by the auditor at boot as the auditor's first-line behavioral specification. They are NOT loaded by the builder, even though they live in the same role file, because the builder's session-start load list excludes the §Audit-Variant section.

This is the lightest-friction asymmetric-corpus mechanism and is required by `auditor-pattern-spec.md`. Adopters cannot opt out of Category D.

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
│   └── (no files — Category D lives in the role file's §Audit-Variant section)
└── E-project-specific/
    ├── INDEX.md
    └── …
```

Category D's directory exists for symmetry and to document the inclusion explicitly; it intentionally contains no files because the materials live in the role file.

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
tags: [<tag>, …]
---
```

The `status` field is the load-time signal: an auditor loading the corpus reads `active` entries normatively, `superseded` and `retracted` entries as historical context (visible but not authoritative), `incorporated` entries as resolved (the post-mortem's lesson is now in the §Audit-Variant clauses; the entry stays for traceability).

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

Post-mortem entries with `status: accepted` should drive concrete changes to the §Audit-Variant clauses or cross-rule obligations. When the change lands, the post-mortem's status flips to `incorporated`. This is the corpus's feedback loop into the role file.

Findings entries can drive Drift catalog entries (Category A → Category C). When three or more findings name the same pattern, the auditor (or a maintainer) authors a Category C entry capturing the pattern, and the contributing Category A entries link forward via `informed: [C-pattern-id]`.

## Access discipline

The corpus's value depends on its asymmetry. The discipline that maintains asymmetry:

1. **Builder MUST NOT load the corpus by default.** The builder's session-start load list (per the role's session-context configuration) does not include any `audit-corpus/` paths. This is the primary discipline.

2. **Auditor MUST load the corpus by default.** The auditor's session-start load list includes the active entries in all populated categories. Superseded and retracted entries are accessible on demand but not loaded eagerly.

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

# Change Control

Update version and provenance on every change.

## Provenance
- source: Initial draft.
- time: 2026-06-06
- summary: v0.1.0 — Initial specification of the audit corpus. Defines five entry categories (prior findings, failure post-mortems, drift catalogs, self-referential §Audit-Variant clauses, project-specific materials), default disk layout, entry frontmatter schema, growth rules (entry, supersession, incorporation), access discipline (asymmetric load, visible but not eager-loaded by builder), bootstrapping guidance for v0.1.0-empty corpora, and anti-pattern catalog. Operationalizes Mechanism 2 of auditor-pattern-spec.md v0.1.0.
