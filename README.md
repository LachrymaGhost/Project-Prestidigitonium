# Project Prestidigitonium

> A domain-agnostic auditor pattern for role-based AI governance systems.

Project Prestidigitonium is a plug-and-play specification for adding an **auditor** alongside any role in a role-based AI governance system — without inheriting the role's blind spots, and without losing the domain fluency that makes audits substantive rather than merely procedural.

The pattern is designed to drop into existing role-based systems by way of a **separate auditor role file** that declares the audited role as a loaded input. No new infrastructure. The auditor knows the audited role's domain because it reads that role's file at boot; the auditor's own rules, posture, and authority live in its own file so the auditor's reasoning isn't bound to the same text as the role it audits.

## What's in this repo

| File | Purpose |
|---|---|
| `auditor-pattern-spec.md` | The canonical specification. Encompassment principle + four mechanisms (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation) + naming + authority + scope limits + integration requirements. |
| `template-auditor-role-file.md` | Copy-pasteable skeleton for the separate auditor role file, with annotated guidance on frontmatter, naming, key-check categories, per-criterion clauses, and cross-rule obligations. |
| `audit-corpus-spec.md` | Structure, growth rules, and access discipline for the asymmetric corpus — the body of materials the auditor loads but the audited role does not. Includes active-learning hooks (forward-compatible v0.1.1 + v0.2.0 location update). |
| `adoption-guide.md` | Step-by-step walkthrough from "role file with rules" to "conformant, correctly-routed, mail-connected auditor alongside it." Includes CLAUDE.md routing strategies and a Migration from v0.1.x section for remaining same-file deployments. |
| `comms-spec.md` | Builder↔auditor mail: message-per-file inboxes, immutable messages with reply-link state, auditor-owned setup (the activation directive bootstraps the builder), self-triggering housekeeping, and **§Reliability** — liveness-by-age, the `.done` cursor-set (no message lost), and the `heartbeat` command surface (`up`/`wait`/`wake`/`down`/`doctor`/`status`/`selftest`) with a deterministic failure-injection suite as its falsification instrument. |
| `conformance-criteria.md` | Twenty-three criteria (seventeen Required, six Recommended) for verifying that an implementation realizes the pattern. Includes verdict structure and review procedure. |
| `CHANGELOG.md` | Version history. |
| `LICENSE` | Apache 2.0. |

## The problem this solves

A role file specifies how a builder should reason about its domain. If a defect lives inside that specification — an unstated assumption, a missing trigger, a conflated category — a reader who treats the specification as authoritative cannot see the defect. The defect is invisible *because* it is the lens through which the reader perceives. Adding an "auditor lens" that reads the same specification with a different intent does not solve this: same words cannot catch what the words do not say.

Naive auditor designs sit at one of two failure modes:

- **Same-file lens** — auditor and builder read the same rules. Domain fluency preserved; correlated blind spots inherited.
- **Fully-independent reviewer** — auditor knows nothing about the role. Blind spots avoided; domain fluency lost.

Project Prestidigitonium resolves the tension via **separate-file architecture with declared inclusion**: the auditor lives in its own role file; the audited role file is declared as a loaded input. Surface familiarity (auditor reads the audited role's file); underlying independence (auditor's own rules and stance live elsewhere).

## The architectural revision (v0.2.0) and retirement of same-file (v0.2.6)

v0.1.x of this specification committed to a *same-file* architecture (audit lens as `# §Audit-Variant` section in the audited role's file). v0.2.0 **inverted** that commitment to **separate files**; v0.2.6 **retired the same-file form entirely** — it is no longer a supported pin point, and the archive branch that preserved it has been removed (git history retains the content). Remaining same-file deployments migrate per `adoption-guide.md` §"Migration from v0.1.x".

The reasoning for the inversion: same-file preserved domain fluency at the cost of shared source text, which is the structural cause of correlated blind spots. Mechanism-level mitigations addressed the symptom; separate-file addresses the cause.

The retirement rests on operational evidence, not just argument: on the first day a separate-file auditor ran, it caught its own author reproducing — in a second auditor file — the exact defect class the author had fixed in the first file hours earlier. Same author, same day, same blind spot; caught only by independent eyes anchored to a different artifact. That is the failure mode same-file architecture cannot catch, observed and remediated in production.

## Key terms (glossary)

- **Builder** — the role being audited. The thing that *does the work*. (In Aire shape: named `<project>-<role>` like `sketch-operator` or `aire-smith`.)
- **Auditor** — the separate auditor role file. Named `<audited-role-slug>-auditor.role.md`; display name `<Project> <Role> Auditor`. The thing that *checks the work*.
- **Audited role file** — the role file the auditor audits. Loaded by the auditor as an input via the `audits:` frontmatter field.
- **Auditor role file** — the auditor's own role file. Has frontmatter `audits:` and `follows_pattern:` declaring the pattern adoption.
- **Reciprocal frontmatter pointers** — the audited role file carries `audited_by:`; the auditor file carries `audits:`. Both pointers required for v0.2.0 conformance.
- **Encompassment** — the structural property that the auditor's knowledge is a proper superset of the builder's. In v0.2.0, made explicit by declared input loading.
- **Asymmetric corpus** — materials the auditor loads that the builder does not. Prior findings, failure post-mortems, drift catalogs, the auditor role file's own body, project-specific asymmetric materials.
- **Adversarial default** — the auditor's epistemic stance. Default verdict: refuted; flips to confirmed only on evidence surviving refutation.
- **Independent re-derivation** — derive-then-compare on key checks. Reading the audited file's claim before deriving is not permitted on these checks.
- **Active-learning hooks** — frontmatter fields and structural conventions that make the corpus programmatically analyzable by future tooling. Documented in v0.1.1 of the corpus spec; extensions deferred until accumulated real corpora provide evidence.
- **Interpretation pinning** — the `audits_version:` frontmatter field recording which version of the audited role file the auditor's per-criterion interpretations were authored against. A session-start mismatch against the audited role's current version flags interpretation staleness before any verdict is issued.
- **Activation directive** — the comms channel's first message, filed by the auditor into the builder's inbox at the auditor's first boot: it teaches the builder the mail protocol through the mail system itself. The builder's session-entry configuration carries only the one-line bootstrap pointer that says to read the inbox.
- **Liveness by age** — the reliability premise (`comms-spec.md` §Reliability): a watch's freshness is judged by the *age* of a heartbeat it writes, never by process-presence — so there is no "who watches the watcher" regress and no instrument to fool. Paired with the `.done` processed-set it yields no-message-lost: *unread* is an order-independent set difference advancing on action, so a missed wake costs latency, never a message.
- **Key-check categories** — the role-specific list of checks that require derive-then-compare (e.g., section presence, frontmatter completeness, governance reference resolution, re-measurement, re-classification).

## Why a separate repository

This specification is governance-pattern material, not project-specific code. It is published as a standalone repository so that:

- It can evolve independently of any one project's release cadence.
- Projects can pin to a specific spec version and upgrade deliberately.
- Adopters across organizations can reference the same canonical pattern.
- It can be cited in role-file provenance as a stable artifact.

This follows the precedent of [Aire-TOC](https://github.com/LachrymaGhost/Aire-TOC) and [Aire](https://github.com/mr-kelley/aire). Project Prestidigitonium composes with both but requires neither — it is additive at a layer they do not occupy.

## How adoption works (v0.2.x)

1. You have a role file. It declares operating rules (or verification checks, or normative requirements).
2. You read `adoption-guide.md` and follow its steps.
3. You copy the skeleton from `template-auditor-role-file.md` into a new file at `<audited-role-slug>-auditor.role.md`.
4. You fill in the placeholders: `audits:` frontmatter pointer, `audits_version:` pin, `audit_posture:`, upstream governance enumeration, asymmetric corpus paths, key-check categories, per-criterion audit interpretations, cross-rule obligations.
5. You add `audited_by: <audited-role-slug>-auditor.role.md` to the audited role file's frontmatter (the only required edit to that file).
6. You create an `audit-corpus/` directory tree at the project root. Empty is fine; v0.1.0-empty is a conformant state.
7. You bump both files' versions and update their provenance.
8. You wire project-harness routing (CLAUDE.md or equivalent) so auditor invocations actually bind to the auditor file — and smoke-test it.
9. You add the one-line mail bootstrap pointer to the builder's session-entry configuration; the auditor's first boot creates the `.comms/` tree and files the activation directive into the builder's inbox — the channel's first message is the builder's operating manual (`comms-spec.md`).
10. You run a conformance check against `conformance-criteria.md`.

Nothing executes. No new infrastructure spins up. The pattern is operational once the auditor file lands, the corpus tree exists, **and** project-harness routing binds auditor invocations to the auditor file. That last condition is the one file-level conformance review cannot see — an adoption with unwired routing looks conformant while every "audit" invocation silently anchors to the builder's own file (adoption guide, Step 8).

## Deployment recommendation: model diversity

The separate-file architecture removes shared **source text** between builder and auditor. It does not remove shared **priors**: a builder and an auditor running on the same model share the same training-shaped reasoning tendencies, and a correlated tendency can survive every file boundary this pattern erects. Deployment can decorrelate what file structure cannot:

1. **Run the auditor on a different model than the builder.** Different model families (or different versions within a family) have different blind spots; the auditor's misses stop correlating with the builder's. This is the primary recommendation.
2. **Capability floor: the auditor's model MUST be at least as capable as the builder's.** A weaker auditor over a stronger builder manufactures the capability blind spots named in `auditor-pattern-spec.md` §"What this pattern does NOT solve." When versions differ, the auditor takes the stronger one — never the reverse.
3. **Elevated thinking tier is a complement, not a substitute.** Running the auditor at a higher reasoning/thinking tier than the builder (e.g., builder at medium, auditor at high) makes the auditor more thorough *within the same lens* — it mitigates sustained-load attention failures, not correlated priors. Use it on top of model diversity when both are available, or as the fallback when only one model is available.
4. **Pin specifics in the adopting project, not here.** Model names age faster than spec versions — naming them normatively in a domain-agnostic spec is an instance of the stale-reference drift class this project catalogs. Adopters SHOULD record their chosen builder/auditor model pairing, with a date, in the auditor role file's Operational Constraints or the project's invocation documentation. (Worked example, as of 2026-06: builder on Claude Opus 4.7, auditor on Claude Opus 4.8 at high thinking.)
5. **Record availability windows when known.** A pinned model with a known access expiry (billing window, preview period) is a staleness with a date. State the window and the planned post-expiry pairing in the clause itself — a known-date staleness declared in advance is a scheduled transition; the same fact left unstated is drift waiting to be discovered by the auditor's boot-time model check.

Like CLAUDE.md routing (Step 8), model selection is invocation-environment configuration — file-level conformance review cannot see it. The obligation to decide it deliberately is part of adoption; the empirical basis is direct: this project's v0.2.3 errata were caught by re-reviewing v0.2.2's artifacts on a different model than the one that authored them.

## Status and stability

This is **v0.5.0**, status **draft**. v0.5.0 evolves the comms watch economy into **§Reliability**: liveness by heartbeat *age* (a passive stamp, never process-presence), a reader-owned processed-set (`.done`) making *unread* an order-independent set difference that advances on action not receipt (no message lost), and a unified self-locating `heartbeat` command surface (`up`/`wait`/`wake`/`down`/`doctor`/`status`/`selftest`) whose reliability is a *falsifiable* property — the deterministic `selftest` failure-injection suite must be identical-GREEN over N runs. Conformance gains C-23 (Recommended). The message protocol is unchanged — an evolution of one layer on a stable spine; commit-signal hooks and the scheduled wake companion are superseded (the companion generalized into the liveness-aware `wake`). v0.4.2 defines `follows_pattern:` pin semantics: the pin declares the *conformed* version and goes stale only when a release changes a conformed surface — ending the every-release re-pin treadmill (whose repair wave had just demonstrably staled its own pins). v0.4.1 advances the template skeleton's `follows_pattern:` example (the recurring maintenance that semantics change retires). v0.4.0 adds the watch-economy provisions to `comms-spec.md` — reader-owned cursors, the watch-lifetime rule (thread-scoped watches bound to a deployed cross-session wake mechanism, as a package), commit-signal hooks with a session-start presence check, and the id-date rule — shaped by an adversarial design review carried over the mail system it revises. v0.3.1 adds the trigger-source provision to the artifact-verdict-only posture (event-triggered verdicts, owner-directed, without posture change). v0.3.0 adds the comms layer (`comms-spec.md`): builder↔auditor mail with auditor-owned setup and housekeeping, the activation-directive bootstrap, and Criterion C-22 (Recommended). v0.2.6 retired the same-file architecture entirely on first-operational-day evidence (see the architectural-revision section) and converted cross-reference fields to pin-free form — versions live in each file and the CHANGELOG, ending the stale-reference treadmill the project's own drift catalog documents. v0.2.4 added the model-diversity deployment recommendation (different model for the auditor, capability floor, thinking-tier elevation as complement). v0.2.5 absorbs the first lessons from running instances: the C-21 multi-file rule-source note, the availability-window deployment point, and the cause-over-output authoring rule for drift catalogs — all three sourced from the RoleSmith Auditor's first operational day. The architectural revision from v0.1.x to v0.2.0 was a breaking inversion under the 0.x convention. v0.2.1 added explicit supersession of v0.1.x architectural language and the CLAUDE.md routing section in the adoption guide. v0.2.2 absorbed Sketch Main Auditor's review of v0.2.0: five errata fixed; new Step 8 in the adoption guide closes the auditor-routing-binding gap; new `audit_posture` declaration mechanism (artifact-verdict-only / continuous-monitoring / hybrid) clarifies that the cold-context discipline scopes to artifact verdicts only. v0.2.3 absorbed a fresh-pass self-review: stale references and counts corrected across the README, adoption guide, and template (the same drift class v0.2.2 fixed, recurring in the release that fixed it — now documented as the seed drift pattern for adopting corpora); `audits_version:` interpretation pinning added (Criterion C-21, Recommended); structured finding schema added to the template's Outputs with a mechanical mapping onto Category A corpus entries; inconclusive-density guidance added to the verdict structure. The pattern is internally consistent and operationally complete — it can be adopted now — but corpus-population conventions will sharpen as real instances accumulate findings and post-mortems.

Versioning policy is in `auditor-pattern-spec.md` §"Versioning."

Production adopters: pin to the commit hash you adopted against. v0.x.y is pre-stable; future minor bumps may introduce new integration requirements.

## Branches

- **`main`** — the separate-file architecture; the only branch. The former `archive/v0.1.x-same-file-architecture` branch was removed at v0.2.6 when the same-file form was retired; its content survives in git history for archaeology, not for adoption.

## Roadmap (informal)

- **v0.2.x (complete)** — clarifications and running-instance lessons. *v0.2.1: v0.1.x supersession language + CLAUDE.md routing section. v0.2.2: Sketch Main Auditor review — 5 errata + Step 8 routing-binding obligation + `audit_posture` declaration mechanism + C-20 Recommended. v0.2.3: fresh-pass errata + `audits_version:` interpretation pinning (C-21 Recommended) + structured finding schema + inconclusive-density verdict guidance. v0.2.4: model-diversity deployment recommendation. v0.2.5: first running-instance lessons — C-21 multi-file rule-source note, availability windows, cause-over-output drift authoring. v0.2.6: same-file architecture retired (archive branch removed); pin-free references convention.*
- **v0.3.0** — the comms layer: `comms-spec.md`, adoption Step 9, Criterion C-22 (Recommended), comms duties in the template skeleton.
- **v0.3.1** — trigger-source provision for the artifact-verdict-only posture.
- **v0.4.0** — watch economy: reader cursors, watch-lifetime rule + scheduled-wake package, commit-signal hooks, id-date rule. *v0.4.1: skeleton pin maintenance. v0.4.2: conformed-version pin semantics.*
- **v0.5.0 (current)** — the reliability layer: `comms-spec.md` §Reliability (liveness-by-age, the `.done` cursor-set, the `heartbeat` command surface, `selftest` as the falsification instrument), Criterion C-23 (Recommended). An evolution of one layer — the message protocol is unchanged; commit-signal hooks and the scheduled wake companion are superseded.
- **v0.4.x** — likely additions: `audits_version:` multi-pin map form (multi-file rule sources); project-level multi-role corpus sharing conventions; explicit cross-corpus pollination protocol; conformance criteria for the specifications themselves (self-conformance, currently reserved).
- **v1.0.0** — declared when at least two independent projects have run the pattern in production for three months and the spec's evolution has stabilized.

## License

Apache License 2.0. See `LICENSE`.

---

*Higitus Figitus Migitus Mum — Prestidigitonium!*
