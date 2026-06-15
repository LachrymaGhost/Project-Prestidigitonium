# Project Prestidigitonium

> A domain-agnostic auditor pattern for role-based AI governance systems.

Project Prestidigitonium is a plug-and-play specification for adding an **auditor** alongside any role in a role-based AI governance system — without inheriting the role's blind spots, and without losing the domain fluency that makes audits substantive rather than merely procedural.

The pattern drops into existing role-based systems as a **separate auditor role file** that declares the audited role as a loaded input. No new infrastructure. The auditor knows the audited role's domain because it reads that role's file at boot; the auditor's own rules, posture, and authority live in its own file, so the auditor's reasoning isn't bound to the same text as the role it audits.

**New here?** Read this page top to bottom for the *why*, then `adoption-guide.md` for the step-by-step *how*. Adopting in an automated/agent workflow? The file table, glossary, and the 10-step adoption section below are written to be machine-followable.

## What's in this repo

| File | Purpose |
|---|---|
| `auditor-pattern-spec.md` | The canonical specification. Encompassment principle + four mechanisms (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation) + naming + authority + scope limits + integration requirements. |
| `template-auditor-role-file.md` | Copy-pasteable skeleton for the separate auditor role file, with annotated guidance on frontmatter, naming, key-check categories, per-criterion clauses, and cross-rule obligations. |
| `audit-corpus-spec.md` | Structure, growth rules, and access discipline for the asymmetric corpus — the body of materials the auditor loads but the audited role does not. Includes active-learning hooks (forward-compatible). |
| `adoption-guide.md` | Step-by-step walkthrough from "role file with rules" to "conformant, correctly-routed, mail-connected auditor alongside it." Includes CLAUDE.md routing strategies and a Migration from v0.1.x section. |
| `comms-spec.md` | Builder↔auditor mail: message-per-file inboxes, immutable messages with reply-link state, auditor-owned setup + self-triggering housekeeping, and **§Reliability** — liveness-by-age, the `.done` cursor-set (no message lost), the `heartbeat` command surface (`up`/`wait`/`wake`/`down`/`doctor`/`status`/`selftest`) with a deterministic self-test as its falsification instrument, and a per-turn unread hook so a busy agent reliably notices waiting mail. |
| `conformance-criteria.md` | Twenty-three criteria (seventeen Required, six Recommended) for verifying an implementation realizes the pattern. Includes verdict structure and review procedure. |
| `CHANGELOG.md` | Full version history. |
| `LICENSE` | Apache 2.0. |

## The problem this solves

A role file specifies how a builder should reason about its domain. If a defect lives inside that specification — an unstated assumption, a missing trigger, a conflated category — a reader who treats the specification as authoritative cannot see the defect. The defect is invisible *because* it is the lens through which the reader perceives. Adding an "auditor lens" that reads the same specification with a different intent does not solve this: same words cannot catch what the words do not say.

Naive auditor designs sit at one of two failure modes:

- **Same-file lens** — auditor and builder read the same rules. Domain fluency preserved; correlated blind spots inherited.
- **Fully-independent reviewer** — auditor knows nothing about the role. Blind spots avoided; domain fluency lost.

Project Prestidigitonium resolves the tension via **separate-file architecture with declared inclusion**: the auditor lives in its own role file; the audited role file is declared as a loaded input. Surface familiarity (the auditor reads the audited role's file); underlying independence (the auditor's own rules and stance live elsewhere).

## Why separate files (the core design choice)

v0.1.x committed to a *same-file* architecture (the audit lens as an `# §Audit-Variant` section inside the audited role's file). v0.2.0 **inverted** that to **separate files**, and v0.2.6 **retired the same-file form entirely** (git history retains it). The reason: same-file preserved domain fluency at the cost of shared source text — the structural cause of correlated blind spots. Mechanism-level mitigations treat the symptom; separate files address the cause.

The retirement rests on operational evidence, not argument alone: on the first day a separate-file auditor ran, it caught its own author reproducing — in a second auditor file — the exact defect class the author had fixed in the first file hours earlier. Same author, same day, same blind spot; caught only by independent eyes anchored to a different artifact. That is precisely the failure same-file architecture cannot catch.

## Key terms (glossary)

- **Builder** — the role being audited; the thing that *does the work*. (In Aire shape: `<project>-<role>`, e.g. `sketch-operator`, `aire-smith`.)
- **Auditor** — the separate auditor role file; the thing that *checks the work*. Named `<audited-role-slug>-auditor.role.md`; display name `<Project> <Role> Auditor`.
- **Audited role file** — the role file the auditor audits, loaded by the auditor via the `audits:` frontmatter field.
- **Reciprocal frontmatter pointers** — the audited file carries `audited_by:`; the auditor file carries `audits:`. Both required for conformance.
- **Encompassment** — the structural property that the auditor's knowledge is a proper superset of the builder's, made explicit by declared input loading.
- **Asymmetric corpus** — materials the auditor loads that the builder does not: prior findings, failure post-mortems, drift catalogs, the auditor's own role file, project-specific asymmetric materials.
- **Adversarial default** — the auditor's stance: default verdict *refuted*, flipping to confirmed only on evidence that survives refutation.
- **Independent re-derivation** — derive-then-compare on key checks; reading the audited file's claim before deriving is not permitted on those checks.
- **Interpretation pinning** — the `audits_version:` field recording which version of the audited file the auditor's per-criterion interpretations target; a session-start mismatch flags interpretation staleness before any verdict.
- **Activation directive** — the comms channel's first message, filed by the auditor into the builder's inbox at first boot: it teaches the builder the mail protocol through the mail system itself.
- **Liveness by age** — the reliability premise (`comms-spec.md` §Reliability): a watch's freshness is judged by the *age* of a heartbeat it writes, never by process-presence — no "who watches the watcher," no instrument to fool. Paired with the `.done` processed-set it yields no-message-lost: *unread* is an order-independent set difference advancing on action, so a missed wake costs latency, never a message.
- **Key-check categories** — the role-specific list of checks that require derive-then-compare (e.g. section presence, frontmatter completeness, governance-reference resolution, re-measurement, re-classification).

## Why a separate repository

This is governance-pattern material, not project-specific code. Publishing it standalone lets it evolve independently of any one project's release cadence; lets projects pin to a spec version and upgrade deliberately; lets adopters across organizations reference the same canonical pattern; and lets it be cited in role-file provenance as a stable artifact. It follows the precedent of [Aire-TOC](https://github.com/LachrymaGhost/Aire-TOC) and [Aire](https://github.com/mr-kelley/aire) — it composes with both but requires neither.

## How adoption works

1. You have a role file declaring operating rules (or verification checks, or normative requirements).
2. Read `adoption-guide.md` and follow its steps.
3. Copy the skeleton from `template-auditor-role-file.md` into `<audited-role-slug>-auditor.role.md`.
4. Fill the placeholders: `audits:` pointer, `audits_version:` pin, `audit_posture:`, upstream governance, asymmetric corpus paths, key-check categories, per-criterion interpretations, cross-rule obligations.
5. Add `audited_by: <audited-role-slug>-auditor.role.md` to the audited role file (the only required edit to that file).
6. Create an `audit-corpus/` tree at the project root. Empty is a conformant state.
7. Bump both files' versions and update their provenance.
8. Wire project-harness routing (CLAUDE.md or equivalent) so auditor invocations bind to the auditor file — and smoke-test it.
9. Add the one-line mail bootstrap pointer to the builder's session-entry config; the auditor's first boot creates `.comms/` and files the activation directive.
10. Run a conformance check against `conformance-criteria.md`.

Nothing executes; no new infrastructure spins up. The pattern is operational once the auditor file lands, the corpus tree exists, **and** harness routing binds auditor invocations to the auditor file — the one condition file-level review cannot see (an unwired adoption looks conformant while every "audit" silently anchors to the builder's own file).

## Deployment recommendation: model diversity

Separate files remove shared **source text**; they do not remove shared **priors**. A builder and auditor on the same model share training-shaped reasoning tendencies, and a correlated tendency can survive every file boundary. Decouple what file structure cannot:

1. **Run the auditor on a different model than the builder** — different families (or versions) have different blind spots, so the auditor's misses stop correlating with the builder's. The primary recommendation.
2. **Capability floor:** the auditor's model MUST be at least as capable as the builder's. A weaker auditor over a stronger builder manufactures capability blind spots. When versions differ, the auditor takes the stronger.
3. **Elevated thinking tier is a complement, not a substitute** — it makes the auditor more thorough *within the same lens* (mitigating sustained-load attention failures), not less correlated. Use it on top of model diversity, or as the fallback when only one model is available.
4. **Pin specifics in the adopting project, not here** — model names age faster than spec versions. Record your builder/auditor pairing, with a date, in the auditor file's Operational Constraints. (Worked example, as of 2026-06: builder on Claude Opus 4.7, auditor on Claude Opus 4.8 at high thinking.)

Like CLAUDE.md routing, model selection is invocation-environment configuration that file-level review cannot see; deciding it deliberately is part of adoption. (Empirical basis: this project's v0.2.3 errata were caught by re-reviewing v0.2.2 on a different model than authored it.)

## Status and stability

**Current: v0.5.2, status: draft.** The pattern is internally consistent and operationally complete — it can be adopted today — and has run in production across multiple projects, accumulating the running-instance lessons that shaped v0.2 through v0.5. Corpus-population conventions will keep sharpening as real instances accumulate findings.

What each release line added (one line each; **full detail in [`CHANGELOG.md`](CHANGELOG.md)**):

- **v0.2.x** — the separate-file architecture; the model-diversity deployment recommendation; the first running-instance lessons. The same-file form was retired on first-operational-day evidence.
- **v0.3.x** — the **comms layer** (`comms-spec.md`): builder↔auditor mail, auditor-owned setup + housekeeping, the activation-directive bootstrap, Criterion C-22.
- **v0.4.x** — the watch economy, plus conformed-version pin semantics (ending the every-release re-pin treadmill).
- **v0.5.x** — the **reliability layer** (Criterion C-23): liveness-by-age; the no-message-lost `.done` cursor; the unified `heartbeat` command surface and its deterministic self-test; the poll collapsed to a uniform low rate; and a per-turn unread hook so a busy agent notices waiting mail even when the activation loop is down. Here reliability is a *falsifiable* property — the self-test must be identical-GREEN over N runs, not merely observed working once.

Versioning policy: `auditor-pattern-spec.md` §"Versioning." **Production adopters:** pin to the commit hash you adopted against; v0.x.y is pre-stable, and future minor bumps may add integration requirements.

## Branches

- **`main`** — the separate-file architecture; the only branch. The former `archive/v0.1.x-same-file-architecture` branch was removed at v0.2.6; its content survives in git history for archaeology, not adoption.

## Roadmap (informal)

- **v0.2.x → v0.4.x (complete)** — separate-file architecture, comms layer, watch economy, conformed-version pin semantics. (See CHANGELOG for the per-release detail.)
- **v0.5.0** — the reliability layer: §Reliability + the `heartbeat` command surface + `selftest`, Criterion C-23.
- **v0.5.1** — collapse the activation poll to a uniform low rate (the adaptive split was dead in production).
- **v0.5.2 (current)** — the per-turn unread hook: a `UserPromptSubmit` backstop surfacing cursor-delta unread every turn, so a busy agent notices mail even when the loop isn't running.
- **Future / backlog** — `audits_version:` multi-pin map (multi-file rule sources); project-level multi-role corpus sharing; cross-corpus pollination protocol; self-conformance criteria (reserved); the wake-on-arrival hook (waking a *live* session mid-task — the harness boundary the reliability layer leaves open).
- **v1.0.0** — declared once at least two independent projects have run the pattern in production for three months and the spec's evolution has stabilized.

## License

Apache License 2.0. See `LICENSE`.

---

*Higitus Figitus Migitus Mum — Prestidigitonium!*
