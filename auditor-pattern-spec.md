---
name: auditor-pattern-spec
version: 0.4.1
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [governance, pattern, audit]
---

# Project Prestidigitonium — Auditor Pattern Specification

## Purpose

This specification defines a domain-agnostic pattern for instantiating an **auditor** alongside a working role (a "builder") in any role-based AI governance system. The pattern is designed so that an auditor can be added to any role with minimal ceremony — by authoring a **separate auditor role file** that declares the audited role as an input — while preserving:

1. **Fluency in the audited role's domain** — the auditor reads the audited role's file as a declared input, so it knows the work the builder does.
2. **Independence from the audited role's blind spots** — the auditor's *own* rules, reading posture, and authority chain live in a separate file, written with the failure modes of the audited role in mind.

The pattern resolves the central tension in same-source auditing: an auditor that shares the builder's source text inherits the builder's blind spots; an auditor that doesn't share the source text loses domain fluency. The resolution is **physical separation with declared inclusion** — the auditor file is separate, but the audited role's file is loaded as input. Surface familiarity (the auditor reads the audited role); underlying independence (the auditor's own rules and stance live elsewhere).

## Why v0.2.0 inverted v0.1.x

v0.1.x of this specification committed to a *same-file* architecture — adding a `# §Audit-Variant` section to the audited role's own file. v0.2.0 inverts that commitment to **separate files**. The reasoning:

- Same-file architecture preserved domain fluency at the cost of shared blind spots: an auditor reading the same words cannot catch what those words do not say.
- Mechanism-level mitigations (asymmetric corpus, adversarial frame, independent re-derivation) addressed the symptom without addressing the structural cause.
- Separate-file architecture addresses the cause directly: the auditor's own rules, posture, and rules live in a different file. The audited role file becomes an input, not the authority.

### Explicit supersession of v0.1.x language

v0.1.x's template carried this architectural note: *"The auditor uses **this same role spec** with an audit lens — there is no separate `*-auditor.role.md` file. The role IS the capability; `audit` is one of two lenses (alongside `execute`) for applying it."* and the v0.1.x adoption guide said adopters who wondered if they needed a separate auditor file *"do not."*

**v0.2.0 reverses both statements.** A separate `<audited-role-slug>-auditor.role.md` file IS now required for conformance (Criterion C-1). Adopters who pinned to v0.1.x reading the confident "no" should understand the project itself has reversed that commitment; the v0.1.x architectural note is hereby superseded for any implementation targeting v0.2.0+. **As of v0.2.6, the same-file architecture is retired entirely**: the archive branch that preserved it has been removed (git history retains the content), and v0.1.x is no longer a supported pin point. Remaining same-file deployments migrate per `adoption-guide.md` §"Migration from v0.1.x", which is retained for exactly that purpose.

### The shift in plain terms

v0.1.x asymmetry was enforced by **load-discipline** — a written instruction that the builder MUST NOT load the §Audit-Variant section even though it lived in the same file the builder otherwise re-read at every anchor. That class of asymmetry is enforced only by instruction-following, which is precisely the failure class the auditor pattern is meant to catch (a reminder, not a gate).

v0.2.0 asymmetry is enforced by the **file boundary**. The auditor file is at a different path; the builder's session-start load list does not reference it; the auditor's corpus and clauses cannot contaminate the builder's context by accident. **The asymmetry shifted from declarative to structural** — from a load-discipline reminder to a file-system gate. This is consistent with the foundations-spec taxonomy: dynamic enforcement supersedes declarative enforcement wherever it can.

The four mechanisms below survive the inversion — they remain load-bearing — but their operationalization moves from "section in the audited role file" to "structure of the separate auditor file." The separate-file architecture is the sole supported form; remaining same-file deployments migrate per the adoption guide.

## Problem statement

A role file specifies how a builder should reason about and act within its domain. If a defect lives inside that specification — an unstated assumption, a missing trigger, a conflated category — a reader who treats the specification as authoritative cannot see the defect; the defect is invisible *because* it is the lens through which the reader perceives. Adding an "auditor lens" that reads the same specification with a different intent does not solve this: same words cannot catch what the words do not say.

This pattern produces auditor instances whose source of truth, reading posture, and asymmetric knowledge access are structured such that builder-side blind spots become visible from the auditor's vantage — without sacrificing the auditor's domain fluency.

## The Encompassment Principle

The auditor's knowledge is a **proper superset** of the builder's:

```
auditor_knowledge ⊇ builder_knowledge
auditor_knowledge \ builder_knowledge ≠ ∅
```

The auditor reads everything the builder reads (the audited role's file, the brief, the domain corpus, the governance specs) **and** material the builder does not load (the asymmetric corpus). The auditor's reading posture and authority chain differ from the builder's even where the corpus overlaps.

In v0.2.0's separate-file architecture, encompassment is *structurally explicit*: the audited role's file is declared as an input in the auditor's frontmatter (`audits:` field) and loaded at session start. The audited role file does not contain the audit lens; the auditor file does. Inclusion-by-input rather than inclusion-by-section.

The four mechanisms below configure encompassment from passive scope expansion into active blind-spot coverage.

## The Four Mechanisms

### Mechanism 1 — Encompassment scope

The auditor's authority chain bottoms out in **upstream governance** (the base role template, kit specs, foundations spec, the project's brief and contract) plus the **asymmetric corpus** (Mechanism 2). The audited role's rules are themselves an audit subject — verifiable against upstream governance — not the final word.

Concretely: if the audited role's file states "X is required" and no upstream governance spec requires X, the auditor is empowered to flag the audited role's rule as the defect rather than confirm the builder's compliance with a possibly-erroneous rule.

This is the substantive content of "the auditor encompasses the role." It does not promote the auditor above the user; the user retains ultimate authority. It promotes the auditor above the audited role file as a reading authority.

### Mechanism 2 — Asymmetric failure-pattern corpus

The auditor reads material that the builder does **not** load by default. This corpus is asymmetric by design and contains:

- **Prior audit findings.** Durable record of defects caught in past sessions, indexed by rule and failure pattern.
- **Failure post-mortems.** Narrative reconstructions of how a defect was missed.
- **Drift catalogs.** Documented patterns of how *this kind of builder* tends to fail.
- **The auditor file's own rule bodies**, which are written *with builder failure modes in mind* — they are auditor-corpus material by design.
- **Gotchas / negative-result documentation** that the builder does not load to avoid biasing toward known-bad patterns by salience.

The asymmetric corpus is the auditor's specialty knowledge: it is what makes the auditor an **expert on how this kind of reasoning fails**. The builder remains expert on *the work*; the auditor adds expertise on *the failure modes of the work*.

The structure, growth rules, and access discipline for the asymmetric corpus are specified in `audit-corpus-spec.md`.

### Mechanism 3 — Adversarial default

The auditor's default verdict is **refuted**. Verdicts flip to **confirmed** only on the basis of evidence that survives an attempt to refute it.

Audit clauses are phrased in terms of what the auditor *attempts to refute*, not what the auditor verifies. The grammatical inversion is load-bearing: the verification frame biases toward confirmation; the refutation frame biases toward catching defects.

The adversarial default is the disciplinary counterweight to the familiarity that encompassment produces. An encompassing auditor that defaults to "this looks fine, I would have done it the same way" provides no independent check.

### Mechanism 4 — Independent re-derivation on key checks

On any check where the auditor could read the builder's output and rubber-stamp it, the auditor is required to **compute its own answer first**, then compare. The check is two-stage: derive, then compare.

Key-check categories that require independent re-derivation:

- **Section presence** — derive from base template what sections MUST appear; then check the artifact.
- **Frontmatter completeness** — derive from base template + role-specific declarations what fields MUST appear; then check the artifact.
- **Relational primitive enumeration** — derive from base template; then check the artifact.
- **Governance embedding requirements** — derive each requirement from its governance spec; then check the artifact's embed.
- **Multi-agent artifact prohibition** (or analogous prohibitions for the project) — derive from normative requirements; then check the artifact.
- **Governance reference resolution** — derive the list of referenced paths from the artifact; then check each path exists.
- **Re-measurement** — for roles producing numeric outputs.
- **Re-classification** — for roles producing categorical outputs.

The category list is role-specific. The auditor file enumerates which categories apply to *its* audit subject.

## Naming convention

Auditor role files are named: **`<audited-role-slug>-auditor.role.md`**.

Examples:
- A role at `aire-smith.role.md` → `aire-smith-auditor.role.md`.
- A role at `sketch-operator.role.md` → `sketch-operator-auditor.role.md`.
- A role at `sketch.role.md` (the Sketch Builder) → `sketch-auditor.role.md`.

Display naming (in headings, prose, conversational reference): `<Project> <Role> Auditor`.

Metaphor language (foreman/boss, twin androids, etc.) may inform design discussion but **does not appear in the artifact**.

## Authority and precedence

Three-level precedence:

1. **User authoritative over auditor verdict.** The auditor logs divergence, surfaces it, and complies; the auditor does not override the user. This is true regardless of the auditor's confidence in its own finding.

2. **Upstream governance authoritative over the audited role's rules.** When the audited role file's rules and upstream governance conflict, upstream governance wins. The auditor flags the audited role's rule as the defect and proposes the upstream-conforming revision (never edits unilaterally).

3. **Audited role's rules authoritative over audit-interpretation phrasing on lens conflicts.** When an audit interpretation could read in a way the audited role's rule body doesn't actually require, the rule body controls — audit interpretations clarify how to verify the rule, they don't extend it.

The asymmetric corpus does not supersede upstream governance; it supplements it with documented failure patterns. Corpus entries that conflict with current governance are flagged as **superseded** rather than overriding.

## Peer reconciliation (multiple decorrelated legs)

The three-level precedence above governs a *single* auditor's verdict. When an adoption runs **more than one decorrelated leg** — two (or more) independent auditors auditing the same builder (a cross-model / cross-vendor triangle), or a builder and auditor reaching different conclusions on a shared question — those legs will sometimes **disagree**. A disagreement between decorrelated legs is a **split**, and how a split is handled is itself load-bearing: mishandled, it destroys the very signal decorrelation exists to produce. The following discipline is **normative for multi-leg adoptions**:

1. **Independent derivation first (unchanged — Mechanism 4).** Each leg derives its own conclusion *before* reading the others'. A split is only meaningful between genuinely independent derivations; a leg that anchors on a peer's verdict has produced no independent signal to reconcile.

2. **A split is debated, negotiated, and triangulated — not immediately escalated.** The legs work the disagreement toward the strongest synthesis: state each position, seek the best path forward, and take inspiration wherever it comes from. Immediately escalating every disagreement to the user throws away the decorrelation dividend (the legs learning by resolving the disagreement) and spends the user's attention prematurely.

   **Safety-class carve-out — fail toward escalation.** One class inverts this default. When a split turns on a **safety, security, legality, or irreversible-harm** boundary — one leg judges that proceeding risks real harm the other would allow — the split is **escalated to the user immediately**, and the debate proceeds *in parallel* with the escalation rather than as a precondition for it. Debate-first is sound because it trades a little latency for the decorrelation dividend; that trade holds only while the cost of a slow resolution is **bounded**. For a safety-class split it is not — the harm can land while the legs are still negotiating — so the polarity flips to **fail toward escalation**: surface first (decision-shaped per (3), both positions and their evidence preserved), *then* keep triangulating toward the strongest synthesis. This is an explicit exception to the **timing** in (2) and (3) only — for a safety-class split the surfacing *precedes* rather than *follows* the debate; the other disciplines are unchanged (it is still never averaged or voted under (4), and a losing safety position is still retained as corpus under (5)). When it is genuinely unclear whether a split is safety-class, it is **treated as safety-class** — the conservative default is the floor: over-escalating a borderline split costs only attention, while under-escalating one can cost the very harm the discipline exists to prevent.

3. **Only what survives honest debate goes to the user — decision-shaped, both positions preserved.** When a split does not resolve through debate, it is surfaced to the user **decision-shaped**: both positions stated in full with their evidence, so the user decides on the merits.

4. **Never averaged, voted, or auto-reconciled (the integrity floor).** A split is **never** resolved by mechanical averaging, majority vote, or silent auto-reconciliation. A one-leg-CONFORMANT vs another-leg-REFUTED split is not noise to smooth away — it is the **decorrelation prize**, the exact signal a single leg could not produce. Averaging destroys it. This floor holds even when the legs *could* reach a tidy compromise; a genuine unresolved split is surfaced intact under (3).

5. **Being wrong is retained, not erased.** A position that loses the debate is kept as **corpus** (an asymmetric failure-pattern entry, Mechanism 2): being wrong teaches what not to do and how to find the right path. A disagreement worked to resolution is evidence, not embarrassment.

This discipline applies identically to **auditor↔auditor peer-triangulation** (the meta-audit surface named under *What this pattern does NOT solve*) and to **builder↔auditor forks**. The **user remains authoritative** throughout (precedence level 1): the discipline governs how a split is *worked and surfaced*, never whether the user's decision binds. *A fuller treatment of multi-auditor concurrency — scheduling, verdict collection, reconciliation records — is a forward spec; this section is the owning statement for the reconciliation discipline itself.*

## File-relationship topology

In the separate-file architecture, three files (and one optional directory) are in play for each adoption:

| File | Authored by | Purpose |
|---|---|---|
| `<audited-role>.role.md` | RoleSmith (or equivalent role authority) | The role being audited. Its frontmatter carries `audited_by: <audited-role>-auditor.role.md` as a reciprocal pointer. No `# §Audit-Variant` section. |
| `<audited-role>-auditor.role.md` | RoleSmith (or equivalent) | The auditor's own role spec. Frontmatter carries `audits: <audited-role>.role.md` and `follows_pattern: Project Prestidigitonium`. Contains the four-mechanism preamble, the auditor's own Operating Rules, the per-criterion audit interpretations for the audited role, and the cross-rule obligations. |
| Upstream governance specs | Project governance authority | Read by both audited role and auditor. The auditor's authority chain anchors here. |
| `audit-corpus/` (optional at v0.1.0-empty) | Auditor maintainers over time | Asymmetric corpus per `audit-corpus-spec.md`. |
| `.comms/` (Recommended, Criterion C-22) | Auditor (setup at first boot + housekeeping) | Builder↔auditor mail per `comms-spec.md`: message-per-file inboxes, immutable messages, reply-link state. The auditor's activation directive bootstraps the builder; the builder's session-entry configuration carries the one-line bootstrap pointer. Mail is not corpus. |

The reciprocal `audits:` ↔ `audited_by:` frontmatter pointers make the relationship discoverable and machine-checkable. Adopters who omit them are non-conformant (Criterion C-1 in `conformance-criteria.md` v0.2.0).

## What this pattern does NOT solve

Honest scope: this pattern resolves correlated blind spots arising from **shared source text**. It does not, by itself, resolve:

- **Capability blind spots** — defects the auditor *could* in principle catch but lacks the technical capability to detect.
- **Adversarial-input failures** — defects induced by inputs deliberately crafted to evade audit. Out of scope for this pattern.
- **Definitional disputes about what constitutes a defect** — the auditor and the user may disagree about whether a finding is real. The pattern routes such disputes to the user; it does not arbitrate them.
- **Sustained-load attention failures** — an auditor that runs continuously may itself drift over long sessions.
- **Meta-audit (auditing the auditor)** — at v0.2.0, the auditor's own role file is not audited by another auditor. The user and peer-triangulation are the meta-audit surfaces; when two decorrelated auditors disagree, *§Peer reconciliation* governs how that split is worked and surfaced (the pattern still routes an unresolved dispute to the user — it does not arbitrate it). Recursive meta-audit chains are out of scope.

These are real limits. Implementations should not claim coverage beyond the pattern's scope.

## Integration requirements

A role-and-auditor adoption is **conformant** with this pattern when it satisfies the following requirements. Detailed verification procedure lives in `conformance-criteria.md` v0.2.0.

1. **A separate auditor role file exists** at `<audited-role>-auditor.role.md` (canonical naming), structured per `template-auditor-role-file.md`, with `audits:` and `follows_pattern:` frontmatter fields populated.

2. **The audited role file declares the pointer** `audited_by: <audited-role>-auditor.role.md` in its frontmatter. Reciprocal discoverability.

3. **The auditor's authority chain is declared** in the auditor file's Normative Requirements: upstream governance + asymmetric corpus, with the user-supreme three-level precedence.

4. **An asymmetric corpus is named** in the auditor file's Inputs — by directory path or convention — and the discipline for what the builder MUST NOT load is stated.

5. **The adversarial default is stated** as the auditor's epistemic posture. Phrasing of audit interpretations follows the refutation frame.

6. **Independent re-derivation requirements are enumerated** in the auditor file's Operating Rules, naming key-check categories for the audited role.

7. **The naming convention is followed**: filename `<audited-role-slug>-auditor.role.md`; display name `<Project> <Role> Auditor`.

8. **The "does not solve" limits are acknowledged** in the auditor file's Scope section, even by reference.

9. **No `# §Audit-Variant` section in the audited role file.** v0.2.0 forecloses the same-file pattern; audited role files contain no audit lens content.

## Versioning

This specification follows semantic versioning.

- **Patch (0.x.y → 0.x.y+1)**: clarifications, examples, typo fixes, non-normative additions.
- **Minor (0.x.y → 0.x+1.0)**: new mechanisms, new integration requirements that do not invalidate existing conformant implementations, new corpus categories.
- **Major (0.x.y → x+1.0.0)**: changes that invalidate existing conformant implementations.

v0.1.x → v0.2.0 was a **breaking revision under the 0.x convention** (same-file → separate-file). Under classical semver (`x.y.z`), a backward-incompatible change pre-1.0.0 is signaled by a minor bump in the 0.x series rather than a major version increment; the README's v0.1.x roadmap warned that minor bumps may invalidate existing implementations during the pre-stable phase. v0.2.0 exercised that warning. v0.2.6 retired the v0.1.x architecture entirely; its content survives only in git history.

Conformant implementations declare which version of this spec they target via their auditor file's frontmatter (`follows_pattern: Project Prestidigitonium v0.2.0` or similar).

## Related patterns

- **Aire foundations + kits** — the architectural substrate this pattern composes with.
- **Aire-TOC** (`https://github.com/LachrymaGhost/Aire-TOC`) — context-routing pattern; complementary at a different layer.
- **Aire base role template** (`claude.role.base.md`) — the role-shape substrate. Both audited and auditor role files derive from it.

This pattern is additive to all three; it does not replace any of them.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Architectural revision of v0.1.x.
- time: 2026-06-07
- summary: v0.2.0 — Inverts the architectural commitment from same-file (audit lens as `# §Audit-Variant` section in the audited role's file) to separate-file (auditor lives in `<audited-role>-auditor.role.md`, with the audited role file declared as an Input via reciprocal `audits:` / `audited_by:` frontmatter pointers). Reasoning: same-file preserved domain fluency at the cost of shared source text — the auditor reading the same words could not catch what those words did not say. Mechanism-level mitigations (asymmetric corpus, adversarial frame, independent re-derivation) addressed the symptom; separate-file addresses the structural cause. The four mechanisms survive the inversion: encompassment scope becomes structurally explicit (audited role declared as input), asymmetric corpus access discipline shifts to loaded by the auditor file alone, adversarial default and independent re-derivation remain unchanged. Integration requirements rewritten around the separate-file topology. Naming convention introduced: filename `<audited-role-slug>-auditor.role.md`; display name `<Project> <Role> Auditor`. v0.1.x implementations remain documented at archive branch `archive/v0.1.x-same-file-architecture`; not invalidated but no longer the canonical direction. Breaking revision under the 0.x convention (the README's v0.1.x roadmap warned minor bumps may invalidate existing implementations during the pre-stable phase). v0.2.1 (2026-06-07) — Adds explicit supersession of v0.1.x architectural language (quoting the v0.1.x template's "no separate file" note and stating its reversal under v0.2.0+) per Sketch Main Auditor's observation that the confident v0.1.x "no" could mislead adopters pinned to v0.1.1. Adds the "asymmetry shifted from declarative to structural" framing — load-discipline reminder replaced by file-system gate — articulating the load-bearing reason the architectural inversion is an improvement rather than just a reorganization. Editorial clarification; no integration-requirement changes from v0.2.0. v0.2.2 (2026-06-07) — Per Sketch Main Auditor's v0.2.0 review (E5): corrects the semver-language contradiction where the spec defined Major as `x+1.0.0` yet labeled 0.1.1→0.2.0 a "major architectural revision" and "major version bump." Defensible under 0.x convention (the README's v0.1.x roadmap explicitly warned that minor bumps may invalidate existing implementations during pre-stable), but the labels contradicted the scheme. Wording corrected to "breaking revision under the 0.x convention" in both the Versioning section and the v0.2.0 provenance entry. No structural changes. v0.2.6 (2026-06-11) — Same-file (v0.1.x) architecture retired entirely by owner decision, on first-operational-day evidence from running instances: a separate-file auditor caught its own author reproducing, in a second auditor file, the exact defect class the author had fixed in the first the same day — the correlated-blind-spot thesis demonstrated on the pattern's designer. The archive branch is removed (content survives in git history); all stay-on-v0.1.x language scrubbed; the migration section in the adoption guide is retained as the path for remaining same-file deployments. Retirement is a support-status change, not a stability claim: the v1.0.0 gate is unchanged. v0.3.0 (2026-06-11) — Comms layer added to the pattern: `.comms/` row in the file-relationship topology per the new `comms-spec.md` (builder↔auditor mail; auditor-owned setup via the activation-directive bootstrap; auditor-owned self-triggering housekeeping; mail-is-not-corpus boundary). Version lockstep at 0.3.0.
- time: 2026-07-16
- summary: v0.4.0 — **Peer reconciliation discipline (multi-leg adoptions).** New `## Peer reconciliation (multiple decorrelated legs)` canonizes how decorrelated legs handle a **split** (a disagreement between independent derivations): (1) independent derivation first (Mechanism 4); (2) debate → negotiate → triangulate, **not** immediate escalation; (3) an unresolved split is surfaced to the user **decision-shaped, both positions preserved**; (4) **never** averaged / voted / auto-reconciled — a CONFORMANT-vs-REFUTED split is the **decorrelation prize**, not noise (the integrity floor); (5) a losing position is retained as **corpus** (Mechanism 2 — being wrong teaches). Applies identically to auditor↔auditor peer-triangulation **and** builder↔auditor forks; the user stays authoritative (precedence level 1). The *does-NOT-solve* peer-triangulation line now points here. **Origin:** canonizes the aire owner's standing ruling (2026-07-16): *"settle it through debate and negotiation, then triangulate the best pathway forward and see inspiration wherever it comes from… it's ok for all of us to be wrong."* Prior state: the split rule lived only in aire memory + correspondence + one auditor seat's cross-rule (an instance, not canon). **Minor bump** (additive — single-auditor adoptions have no peer, so none is invalidated). Umbrella README → v0.7.0 (roadmap line added). **Forward:** a multi-auditor concurrency + verdict-reconciliation spec (scheduling / collection / reconciliation records) and a conformance criterion elaborate this section. Authored by RoleSmith (aire-smith, Opus 4.8) for the aire cross-vendor triangle; **verified by peer-triangulation on the reconciliation rule itself** — aire-smith-auditor (Fable 5) + the Gemini cross-vendor auditor derive independently; owner adopts. PP working-tree edit (LachrymaGhost/Project-Prestidigitonium); committed to a branch after owner adoption, never mr-kelley. v0.4.1 (2026-07-16) — **Safety-class carve-out.** §Peer reconciliation point (2) gains a carve-out: a split turning on a **safety / security / legality / irreversible-harm** boundary **fails toward escalation** — it is surfaced to the user *immediately* and the debate proceeds *in parallel* rather than as a precondition, an explicit exception to the **timing** of (2) and (3) (surface precedes debate); disciplines (4) never-averaged and (5) losing-position-as-corpus are unchanged. Rationale: debate-first trades latency for the decorrelation dividend, sound only while the cost of a slow resolution is **bounded** — a safety-class split's is not (harm lands during negotiation), so the polarity inverts. Includes the conservative default: an ambiguous-class split is **treated as safety-class** (over-escalation costs only attention; under-escalation can cost the harm). **Origin:** the aire 3-party stability triangulation (2026-07-16) — both the Fable auditor (N2) and the Gemini cross-vendor auditor independently found the v0.4.0 canon **silent on safety-class splits**; convergent finding, not a single leg's. Runs the §Peer-reconciliation discipline on its own genesis (the carve-out is itself the product of a peer-triangulated split-finding). Inline **test simulation** passed before landing (per the aire test-simulation gate): closes the harm-during-negotiation gap; no debate-first regression for ordinary verdict splits (class is enumerated); integrity floor + point-3 timing-exception verified coherent. **Minor bump** (additive — narrows point (2) for one class; no existing adoption invalidated, single-auditor adoptions have no peer). Umbrella README → v0.7.1. Authored by RoleSmith (aire-smith, Opus 4.8); **delta-verified by peer-triangulation** — aire-smith-auditor (Fable 5) + Gemini cross-vendor auditor derive independently; owner adopts. Same push discipline (branch after adoption, never mr-kelley).
