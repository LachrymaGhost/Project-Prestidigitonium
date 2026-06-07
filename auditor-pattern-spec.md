---
name: auditor-pattern-spec
version: 0.1.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [governance, pattern, audit]
---

# Project Prestidigitonium — Auditor Pattern Specification

## Purpose

This specification defines a domain-agnostic pattern for instantiating an **auditor** alongside a working role (a "builder") in any role-based AI governance system. The pattern is designed so that a single template can be plug-and-played into any role file to produce a paired auditor that is simultaneously:

1. **Fluent in the role's domain** — enough to recognize substantive defects, not merely procedural ones.
2. **Independent enough to catch correlated blind spots** — not merely re-reading the same rules and confirming the builder's interpretation.

The pattern resolves the central tension in same-source auditing: an auditor that shares the builder's source text inherits the builder's blind spots; an auditor that doesn't share the source text loses domain fluency. Both failure modes are common, and naive same-file audit lenses produce the first while naive fully-independent reviewers produce the second.

## Problem statement

A role file specifies how a builder should reason about and act within its domain. If a defect lives inside that specification — an unstated assumption, a missing trigger, a conflated category — a reader who treats the specification as authoritative cannot see the defect; the defect is invisible *because* it is the lens through which the reader perceives. Adding an "auditor lens" that reads the same specification with a different intent does not solve this: same words cannot catch what the words do not say.

This pattern produces auditor instances whose source of truth, reading posture, and asymmetric knowledge access are structured such that builder-side blind spots become visible from the auditor's vantage — without sacrificing the auditor's domain expertise.

## The Encompassment Principle

The auditor's knowledge is a **proper superset** of the builder's:

```
auditor_knowledge ⊇ builder_knowledge
auditor_knowledge \ builder_knowledge ≠ ∅
```

The auditor reads everything the builder reads (shared domain corpus) **and** material the builder does not load (asymmetric corpus). The auditor's reading posture and authority chain differ from the builder's even where the corpus overlaps.

This is the **structural** version of the principle. Its expression in the role file is the **§Audit-Variant** section (see `template-audit-variant-section.md`), which encodes the auditor's interpretation of each operating rule alongside cross-rule obligations that bind the auditor across the rule set.

Encompassment alone is necessary but not sufficient. The auditor must also be configured by the four mechanisms below to convert encompassment from passive scope expansion into active blind-spot coverage.

## The Four Mechanisms

### Mechanism 1 — Encompassment scope

The auditor's authority chain bottoms out in **upstream governance** (the base role template, kit specs, foundations spec, the project's brief and contract) plus the **asymmetric corpus** (Mechanism 2). The role file's rules are themselves an audit subject — verifiable against upstream governance — not the final word.

Concretely: if the role file states "X is required" and no upstream governance spec requires X, the auditor is empowered to flag the role file's rule as the defect rather than confirm the builder's compliance with a possibly-erroneous rule.

This is the substantive content of "the auditor encompasses the role." It does not promote the auditor above the user; the user retains ultimate authority. It promotes the auditor above the role file as a reading authority.

### Mechanism 2 — Asymmetric failure-pattern corpus

The auditor reads material that the builder does **not** load by default. This corpus is asymmetric by design and contains:

- **Prior audit findings.** Durable record of defects caught in past sessions, indexed by rule and failure pattern.
- **Failure post-mortems.** Narrative reconstructions of how a defect was missed, what reasoning produced the miss, and what would have caught it earlier.
- **Drift catalogs.** Documented patterns of how *this kind of builder* tends to fail (e.g., for execution-heavy roles: optimism in interpretation, tunnel vision under iteration pressure; for authoring-heavy roles: silently dropping requirements, citing deprecated references).
- **The §Audit-Variant clauses themselves**, which are written *with builder failure modes in mind* — they are auditor-corpus material whether or not we name them as such.
- **Gotchas / negative-result documentation** that the builder does not load to avoid biasing toward known-bad patterns by salience.

The asymmetric corpus is the auditor's specialty knowledge: it is what makes the auditor an **expert on how this kind of reasoning fails**. The builder remains expert on *the work*; the auditor adds expertise on *the failure modes of the work*.

The structure, growth rules, and access discipline for the asymmetric corpus are specified in `audit-corpus-spec.md`.

### Mechanism 3 — Adversarial default

The auditor's default verdict is **refuted**. Verdicts flip to **confirmed** only on the basis of evidence that survives an attempt to refute it.

This is an epistemic stance, not a procedure. Audit clauses are phrased in terms of what the auditor *attempts to refute* (e.g., "the auditor attempts to refute the claim that this completion is evidence-based"), not what the auditor verifies. The grammatical inversion is load-bearing: the verification frame biases toward confirmation; the refutation frame biases toward catching defects.

The adversarial default is the disciplinary counterweight to the familiarity that encompassment produces. An encompassing auditor that defaults to "this looks fine, I would have done it the same way" provides no independent check. The adversarial default forecloses that drift by requiring evidence to *clear* the verdict, not evidence to *raise* a flag.

### Mechanism 4 — Independent re-derivation on key checks

On any check where the auditor could read the builder's reasoning and rubber-stamp it, the auditor is required to **compute its own answer first**, then compare. The check is two-stage: derive, then compare. Reading the builder's output before deriving is not permitted on these checks.

Key-check categories that require independent re-derivation:

- Independent **re-measurement** of any reported numeric value (for execution roles).
- Independent **re-classification** of any decision class (Class A / B / C).
- Independent **re-derivation** of what acceptance criteria mean in the current context.
- Independent **rule-applicability mapping** — which rules govern this work, derived from the brief and governance, not from the builder's invocation.
- Independent **completeness check** — what *should* be present, derived from upstream specs, regardless of what the builder produced.

Independent re-derivation is the disciplinary counterweight to "I would have made the same call." It refuses the auditor the comfort of reading the answer before forming its own.

## Naming convention

Auditor instances are named functionally: **`<Project> <Role> Auditor`**.

Examples:
- A renderer project named "Sketch" with an Operator role → **Sketch Operator Auditor**.
- A role-authoring project (e.g., Aire) with a RoleSmith role → **RoleSmith Auditor** (project name implicit when self-evident).

The name says what the auditor does. Metaphor language (foreman/boss, twin androids, etc.) may inform design discussion but **does not appear in the artifact**.

## Authority and precedence

When the §Audit-Variant clauses and the execute-mode rule bodies appear to conflict, **the execute-mode rule body is authoritative**. The §Audit-Variant clauses are the auditor's interpretation of how to verify compliance with the execute-mode rules; they are not a separate, independent obligation.

When the role file's rules and upstream governance appear to conflict, **upstream governance is authoritative** (Mechanism 1). The auditor flags the role file's rule as the defect and proposes the upstream-conforming revision.

When the auditor's verdict and the user's directive appear to conflict, **the user is authoritative**. The auditor logs the divergence, surfaces it, and complies; the auditor does not override the user. This is true regardless of the auditor's confidence in its own finding.

The asymmetric corpus does not supersede upstream governance; it supplements it with documented failure patterns. Corpus entries that conflict with current governance are flagged as **superseded** rather than overriding.

## What this pattern does NOT solve

Honest scope: this pattern resolves correlated blind spots arising from **shared source text**. It does not, by itself, resolve:

- **Capability blind spots** — defects the auditor *could* in principle catch but lacks the technical capability to detect (e.g., a measurement instrument the auditor cannot re-invoke). Mitigated by Mechanism 4 but not eliminated.
- **Adversarial-input failures** — defects induced by inputs deliberately crafted to evade audit. Out of scope for this pattern; addressed by separate security review disciplines.
- **Definitional disputes about what constitutes a defect** — the auditor and the user may disagree about whether a finding is real. The pattern routes such disputes to the user; it does not arbitrate them.
- **Sustained-load attention failures** — an auditor that runs continuously may itself drift over long sessions. Anchor beacons and session-restart discipline mitigate; the pattern does not eliminate.

These are real limits. Implementations should not claim coverage beyond the pattern's scope.

## Integration requirements

A role file is **conformant** with this pattern when it satisfies the following requirements. Detailed verification procedure lives in `conformance-criteria.md`.

1. **A §Audit-Variant section exists** in the role file, structured per `template-audit-variant-section.md`, with one audit-interpretation clause per operating rule (or verification check) plus a minimum of three cross-rule audit obligations.

2. **The auditor's authority chain is declared** in the §Audit-Variant preamble: upstream governance + asymmetric corpus, with execute-mode rule bodies authoritative on lens conflicts.

3. **An asymmetric corpus is named** — by file, directory, or convention — and a discipline is stated for what the builder MUST NOT load. (See `audit-corpus-spec.md` for the corpus's internal structure.)

4. **The adversarial default is stated** as the auditor's epistemic posture. Phrasing of audit clauses follows the refutation frame.

5. **Independent re-derivation requirements are enumerated** for the role's key-check categories. The role file states which categories require derive-then-compare and which permit read-then-verify.

6. **The naming convention is followed**: the auditor instance is referred to as `<Project> <Role> Auditor` in the role file's §Audit-Variant heading and any cross-references.

7. **The "does not solve" limits are acknowledged** in the §Audit-Variant section, even by reference. Implementations that silently claim coverage beyond the pattern's scope are non-conformant.

## Versioning

This specification follows semantic versioning.

- **Patch (0.x.y → 0.x.y+1)**: clarifications, examples, typo fixes, non-normative additions.
- **Minor (0.x.y → 0.x+1.0)**: new mechanisms, new integration requirements that do not invalidate existing conformant implementations, new corpus categories.
- **Major (0.x.y → x+1.0.0)**: changes that invalidate existing conformant implementations (e.g., renaming a mechanism, restructuring the §Audit-Variant skeleton).

Conformant implementations declare which version of this spec they target.

## Related patterns

- **Aire foundations + kits** — the architectural substrate this pattern composes with. An auditor's asymmetric corpus is naturally implemented as a kit-style module.
- **Aire-TOC** (`https://github.com/LachrymaGhost/Aire-TOC`) — the context-routing pattern; complementary at a different layer. TOC routes which context surfaces; this pattern governs how a particular role's outputs are audited within whatever context is routed.
- **Aire base role template** (`claude.role.base.md`) — the role-shape substrate. §Audit-Variant sections are added to roles derived from this template.

This pattern is additive to all three; it does not replace any of them.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Initial draft.
- time: 2026-06-06
- summary: v0.1.0 — Initial specification of the auditor pattern. Establishes encompassment principle, four mechanisms (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation), naming convention, authority precedence, scope limits, integration requirements, and versioning conventions. Domain-agnostic; intended for plug-and-play instantiation in any role-based AI governance system. References Aire foundations+kits, Aire-TOC, and Aire base role template as related but non-required substrates.
