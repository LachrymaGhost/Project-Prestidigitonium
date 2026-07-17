# Changelog

All notable changes to Project Prestidigitonium are documented here. Versioning follows semantic versioning per `auditor-pattern-spec.md` §"Versioning."

## [0.7.1] — 2026-07-16

### Origin
The **3-party stability triangulation** that followed v0.7.0 (builder + Claude auditor + cross-vendor Gemini auditor asking "are we ready for real work?") surfaced that the freshly-canonized §Peer reconciliation was **silent on safety-class splits**. Both auditors found it **independently** — the Fable auditor as note N2 ("fail-toward-escalation should explicitly outrank debate-first there — one clause closes it") and the Gemini cross-vendor auditor concurrently ("a safety-split exception needs to be added to the peer-discipline canon"). A convergent cross-vendor finding, not a single leg's — the discipline caught a gap in its own genesis.

### Added
- **`auditor-pattern-spec.md` v0.4.0 → v0.4.1 — safety-class carve-out in §Peer reconciliation point (2).** A split turning on a **safety / security / legality / irreversible-harm** boundary **fails toward escalation**: it is surfaced to the user *immediately* and the debate proceeds *in parallel* rather than as a precondition. The rationale is explicit — debate-first trades latency for the decorrelation dividend, sound only while the cost of a slow resolution is **bounded**; a safety-class split's is not (the harm can land while the legs negotiate), so the polarity inverts. An **explicit exception to the timing of (2) and (3) only** (the surfacing precedes the debate); disciplines **(4) never-averaged** and **(5) losing-position-as-corpus** are unchanged. Includes the conservative default: an ambiguous-class split is **treated as safety-class** — over-escalation costs only attention, under-escalation can cost the harm the discipline exists to prevent.

### Retained unchanged
The comms/reliability surface, the message protocol, and the **auditor-role-file conformance surface (C-1..C-23)** are untouched. This release is **additive** — it narrows point (2) for one class of split and invalidates no existing adoption (a single-auditor adoption has no peer). Existing `follows_pattern:` pins stay conformant.

### Verification
Authored by the aire RoleSmith (aire-smith, Opus 4.8). An inline **test simulation** passed before landing (per the aire test-simulation gate): the carve-out closes the harm-during-negotiation gap, introduces no debate-first regression for ordinary verdict splits (the safety class is enumerated, not "any high-stakes split"), and is coherent with the integrity floor and point-3 timing. **Delta-verified by peer-triangulation** — the Claude auditor (Fable 5) and the Gemini cross-vendor auditor derive independently; owner adopts. Committed to a LachrymaGhost/Project-Prestidigitonium branch after adoption, never mr-kelley.

## [0.7.0] — 2026-07-16

### Origin
Running an adoption with **two decorrelated auditors** (the aire cross-vendor triangle: builder + a Claude auditor + a cross-vendor Gemini auditor) surfaced that the pattern named *peer-triangulation* as a meta-audit surface but never said **how two decorrelated legs reconcile a disagreement**. The rule lived only in one deployment's memory + correspondence + a single auditor seat's cross-rule — not in canon. The aire owner ruled it (2026-07-16): *"settle it through debate and negotiation, then triangulate the best pathway forward and see inspiration wherever it comes from… it's ok for all of us to be wrong — being wrong helps us understand what not to do."*

### Added
- **`auditor-pattern-spec.md` v0.3.0 → v0.4.0 — new `## Peer reconciliation (multiple decorrelated legs)`.** The owning statement for how decorrelated legs handle a **split**: (1) independent derivation first (Mechanism 4); (2) **debate → negotiate → triangulate, not immediate escalation**; (3) an unresolved split surfaced to the user **decision-shaped, both positions preserved**; (4) **never averaged / voted / auto-reconciled** — a CONFORMANT-vs-REFUTED split is the **decorrelation prize**, not noise (the integrity floor); (5) a losing position retained as **corpus** (Mechanism 2 — being wrong teaches). Applies to auditor↔auditor peer-triangulation **and** builder↔auditor forks; the user stays authoritative (precedence level 1). The *does-NOT-solve* peer-triangulation line now points to it.

### Retained unchanged
The comms/reliability surface, the message protocol, and the **auditor-role-file conformance surface (C-1..C-23)** are untouched — this release is **additive** (a single-auditor adoption has no peer, so none is invalidated). Existing `follows_pattern:` pins stay conformant (C-1 conformed-version semantics; no re-pin treadmill).

### Forward
A **multi-auditor concurrency + verdict-reconciliation spec** (scheduling, verdict collection, reconciliation records) and a matching conformance criterion elaborate this section — named, not yet built.

### Verification
Authored by the aire RoleSmith (aire-smith, Opus 4.8). **Verified by peer-triangulation on the reconciliation rule itself** — the Claude auditor (Fable 5) and the Gemini cross-vendor auditor derive independently; owner adopts. Committed to a LachrymaGhost/Project-Prestidigitonium branch after adoption, never mr-kelley.

## [0.6.2] — 2026-06-25

### Origin
A **four-frame cross-implementation triangulation** of the v0.6.0 armed-listening feature — this leg's builder↔auditor pair, an external decorrelated aire pair, and a sibling DT-pair auditor, judged against the spec text + three independent v0.6.0 builds — found the armed-listening under-pinned the **session-id source** and its **absent** behaviour; and a parallel watcher-bug investigation found a `/compact`-orphaned waiter **masks `armed`** (it shares the preserved session id and passes the live-task check). "The reference impl is immune" was itself **refuted** — it shared the fail-open class. Decorrelation at the ecosystem level: three independent frames converging on one spec gap.

### Changed
- **`comms-spec.md` v0.6.0 → v0.6.2 — §Armed-listening hardened** (a tightening of v0.6.0's armed-listening; v0.5.x untouched). (1) own-session armed MUST **fail-CLOSED** on an absent/empty session source — never a freshness-only or process-liveness floor (the silent-degrade named as the anti-pattern); (2) the session-id source **pinned by property** (guaranteed-present, changes-on-new-session, harness-provided), explicitly **not named** in the spec; (3) two binding-blind boundaries named — a context compaction preserves the id, a sub-agent shares its parent's id; (4) the **live-task check promoted from prose discipline to a coded provision** (the marker carries the wake's pid; armed requires that process alive — GP2 / enforce-by-mechanism); (5) the cross-leg readiness path made **explicit / opt-in** (`--cross-leg` freshness-floor), never the silent own-session fallback; (6) a **single-waiter identity (singleton) provision** — the live-task check is necessary-but-NOT-sufficient: a context-reset-orphaned waiter passes liveness yet masks `armed`, so the wake must be a per-slug singleton that retires the orphan on arm; (7) §falsifiable-property gains absent-source→fail-closed, dead-wake→armed=no, and a **duplicate/orphan-waiter injection**.
- **`comms-bringup-directive.md` v2.3 → v2.4** — the `heartbeat gate`/`doctor` **version-skew conditional** (`gate` is a v0.6.0+ command; a pre-armed-listening `bin/` uses `doctor`), stated by pointing to the adopted version (no surface re-enumeration, per C-23's de-enumeration) + reflects the v0.6.2 hardening. No new gate; Gate #7's proof unchanged.
- **`conformance-criteria.md` v0.3.3 → v0.3.4 — C-23 failure-mode conjunct.** v0.3.3 closed the command-surface gap; this closes it one level down — C-23 is command-surface-keyed, so a happy-path `selftest` could pass while omitting the new behaviour cases. C-23 now ALSO requires the suite **inject each failure mode the adopted `comms-spec.md` §falsifiable-property enumerates** — by pointing to that section, not copying (self-updating: the orphan-waiter injection is auto-covered). Backward-conformance preserved by C-1; C-23 stays Recommended.

### Reference implementation (the Aire deployment — built + proven, not part of the spec)
`comms-lib.sh` / `comms-wait` / `comms-doctor` implement the above: the singleton-claim (exact-arg `/proc/cmdline` reap) + EXIT-trap (both **PID-guarded** so a reaped orphan can't clobber the current waiter) + the live-task check + fail-closed; the wake-marker carries the stamping waiter's pid. `heartbeat selftest` is **identical-GREEN 50/0 across N runs** incl. the orphan-injection. Decorrelated-auditor-GREEN before landing (proven-then-written).

### Retained unchanged
`auditor-pattern-spec.md`, `audit-corpus-spec.md`, the message protocol, and the **auditor-role-file conformance surface (C-1..C-17)** are untouched — so existing auditor-role-file `follows_pattern:` pins (v0.6.1) stay conformant: this release moves the **comms-reliability** surface only, and per C-1's conformed-version semantics a pin older than HEAD is conformant when no conformed surface it depends on moved (no re-pin treadmill).

## [0.6.1] — 2026-06-24

### Fixed (errata)
- **`conformance-criteria.md` v0.3.3 — C-23 de-enumeration (corrects a stale criterion shipped in v0.6.0).** C-23's "what is checked" hardcoded the reference command list (`heartbeat up|wait|wake|down|doctor|status|selftest`), which **omitted the v0.6.0 `heartbeat gate`** and would need hand-syncing on every surface growth. It now points to **the adopted `comms-spec.md` version's command-surface contract**, and its `selftest` conjunct is tied to **exercise that adopted surface in full** — closing the gap whereby a v0.6.0 adopter could pass C-23 with a gate-less surface and a GREEN `selftest` that never exercised `gate`. A command from the adopted surface that is absent **or present-but-unexercised** is now an explicit Failure condition (previously C-23 failed on no missing command at all). One-home-per-rule restored: the surface lives once in `comms-spec.md`; C-23 points, it does not copy — retiring the drift surface itself (per the `references:` pin-free [0.2.6] and conformed-version [C-1] precedents).
- **Errata to the v0.6.0 "no criteria change" claim.** The v0.6.0 "Retained unchanged" note asserted C-23 "already covers" the wake-marker/`gate` with "no criteria change." That was an **overclaim**: C-23 required the `selftest` GREEN, not that it *exercise* `gate`, so a gate-less v0.6.0 adopter could pass — coverage the criterion text did not guarantee. v0.6.1 makes the coverage real. The v0.6.0 *mechanism* (the wake-marker + `gate`) was correct and is unchanged; only the **conformance criterion** that should have gated it is fixed here.
- **Backward-conformance preserved (conformed-version semantics, C-1).** A v0.5.x-§Reliability adopter's "adopted version's surface" is v0.5.x — it owes no `gate`; only v0.6.0+ adopters do. No existing instance is retroactively non-conformant; C-23 remains **Recommended**; no Required criterion changed.

### Changed
- **`README.md`** — Status/Roadmap synced to v0.6.1 (the C-23 errata line).
- **`adoption-guide.md` v0.3.4 → 0.3.5 (Step 6) + `template-auditor-role-file.md` v0.3.6 → 0.3.7** — the example `follows_pattern:` pins advance v0.5.0 → v0.6.1: v0.6.1 is the release where the comms-reliability command surface becomes a **conformed** surface (C-23 v0.3.3 now requires it), so the example pin advances per the conformed-surface-moved pin semantics (C-1) — *not* tracks-latest (v0.6.0, where the mechanism merely appeared with no criterion requiring it, is skipped). The same lockstep applied when comms duties moved at v0.5.0; each file bumps its own version + provenance.

### Origin
A **cross-project decorrelation catch.** While a sibling project's decorrelated auditor reviewed its own plan to adopt PP v0.6.0, it read C-23, could not find `gate`, and routed the gap "upstream to PP." The pattern's own builder↔auditor pair then reproduced and verified it refute-default before this errata — decorrelation operating at the ecosystem level: one project's auditor catching a published gap another pair missed at release. (Method note: the v0.6.0 "no criteria change" line is preserved verbatim in its original [0.6.0] entry and corrected here as dated errata — not rewritten in place — per the [0.2.3] errata precedent, so the record of the claim stays auditable.)

## [0.6.0] — 2026-06-23

### Origin
A decorrelated builder↔auditor pair running this pattern in production took the comms reliability layer one iteration past v0.5.4's autonomous-wake gate, on the operator directive "make spin-up 100% reliable, no per-spin-up diagnosis." Root insight (the pair's shared-blind-spot pass): **liveness-by-age proves the WATCH alive, not the agent-waking WAKE armed** — `heartbeat up` brings the watch up without arming the wake (the *up ≠ armed* seam), and the heartbeat goes GREEN regardless of agent state (the *GREEN ≠ listening* gap). v0.5.4's directive even **named** the residual ("an event-wake that quietly died is indistinguishable from no mail") and left re-arming to discipline. This release closes it by mechanism. Built + selftest-proven + verified decorrelated (the auditor ran the production enacting path by hand and confirmed the binding live) before landing — **proven-then-written**.

### Added
- **`comms-spec.md` v0.6.0 — §Reliability: the wake-marker (armed-listening).** A new liveness mechanism (minor bump per `auditor-pattern-spec.md` §Versioning; **additive under conformed-version-pinning** — a v0.5.x-pinned instance is not retroactively non-conformant; a v0.6.0 *adopter* does provide the new `heartbeat gate` + wake-marker surface). The armed wake stamps a tight-freshness, **session-bound** marker (`<run>/<slug>.wake`) every poll, so **armed-listening ≡ fresh + session-bound** — as falsifiable as liveness. A quietly-died / never-armed / other-session wake reads **`armed=no`** (the named residual, now detectable). New **`heartbeat gate <slug>`** asserts **live + armed + drained** → READY / NOT-READY before "boot complete" (session-scoped; cross-leg readiness uses the freshness-floor). Folded reference-impl hardening: `cb_mark_processed` rejects an empty id (the silent drain no-op); the display cursor labeled lexical-max / display-only (authoritative unread = `cb_unread_count`). Reliability backed by its falsification instrument: `heartbeat selftest` extended to **43 checks** incl. the production enacting path (omitted-arg session binding), the gate's armed-fail path, and the freshness bracket — identical-GREEN.
- **`comms-bringup-directive.md` v2.3** — armed-listening folded into §4b + the Standing health check: `heartbeat gate` as the continuous (re-runnable) form of the setup-time Gate #7, the per-turn boot-incomplete cue, and a SessionStart auto-bring-up convenience (firing verified-per-deployment, floored by the per-turn surface). Adds no new gate.

### Changed
- **`README.md`** — front-door synced to v0.6.0 (status + the comms reliability line).

### Retained unchanged
- The message protocol, `auditor-pattern-spec.md`, and `audit-corpus-spec.md` are untouched. `conformance-criteria.md` **C-23** (comms reliability provisions adopted and proven) already covers the wake-marker under "the reference command surface is present + `heartbeat selftest` reproducibly identical-GREEN" — the new `gate` / wake-marker selftest cases fall under it; **no criteria change**. SessionStart auto-bring-up is a deployment convenience whose firing is verified per-deployment — **not** claimed as a proven spec mechanism; the per-turn `UserPromptSubmit` surface is the proven floor.

## [0.5.4] — 2026-06-18

### Origin
Owner directive after a decorrelated audit pair, running this pattern in production, hit two comms bring-up failures in one engagement: (1) a per-turn unread banner mis-scoped in a shared checkout silently covered only one leg, and (2) a heads-down `/loop` leg relying on a ~20-minute fallback wake with no inbox event-monitor missed mail that arrived in the dead window between fires. Both were operator/bring-up failures, not mechanism defects — the fix is to make the bring-up *prove itself*, including the autonomous-wake path the reliability layer leaves to the harness. Authored by the running auditor; the v2.2 autonomous-wake provisions were drafted by the builder leg and verified decorrelated (refute-default) by the auditor before landing.

### Added
- **`comms-bringup-directive.md`** — the ordered, idempotent, gate-verified procedure for standing the builder↔auditor comms channel up **correct the first time**, the operational expansion of `adoption-guide.md` Step 9. Five root-cause failure modes (copy-from-instance; no end-to-end gate; shared-checkout scoping; two-party coordination; **no autonomous-wake**), a two-party bring-up sequence, and a **seven-check Mandatory Verification Gate** that proves the channel *delivers, surfaces per-turn, gets read/replied, advances a cursor, AND wakes a heads-down agent* before work begins. Declaring setup "done" on file-existence is the recurring failure; the gate replaces that with proof.
- **The autonomous-wake operationalization (directive §4b + Gate #7).** The per-turn unread surface (`comms-spec.md` §Activation backstop) fires only on a *submitted turn*; an autonomous/heads-down/loop agent submits none, so its only wake is whatever timer it set, and a long timer alone drops mail into its dead window. §4b directs arming an **event-wake on the agent's own inbox** — the host wake-on-arrival primitive where it exists, `heartbeat wait` where the deployment runs it, a bounded fallback poll behind both — and **Gate #7** proves it actually re-invokes a heads-down agent within bounded latency (not at the next long-timer fire), or records the latency bound explicitly where only the fallback exists. This **operationalizes, at the deployment layer, the wake-on-arrival hook `comms-spec.md` §"does not cover" leaves to the harness** — it adds **no spec mechanism** and no conformance criterion.

### Changed
- **`README.md`** — front-door synced to v0.5.4: new file-table row for `comms-bringup-directive.md`; Step 9 (How adoption works) references the directive for a correct-first-time bring-up; Status section bumped + a v0.5.4 release-line note; Roadmap notes that the directive's §4b/Gate #7 now operationalizes and verifies the wake-on-arrival hook at the deployment layer (a host-agnostic spec mechanism remains open).
- **`adoption-guide.md` v0.3.4** — Step 9 references `comms-bringup-directive.md` as the correct-first-time bring-up path (the directive operationalizes Step 9). No step-structure or criteria changes.

### Retained unchanged
- No conformance-criteria, message-protocol, or spec-mechanism changes. `comms-spec.md`, `auditor-pattern-spec.md`, `conformance-criteria.md`, and `template-auditor-role-file.md` are untouched; all v0.5.x channels and existing instances remain conformant (C-22/C-23 unaffected). The directive is an operational companion that *sequences and verifies* existing provisions; the wake-on-arrival hook it operationalizes remains, as a spec mechanism, explicitly out of `comms-spec.md` (a deployment/harness concern).

## [0.5.3] — 2026-06-15

### Origin
Owner directive after a running auditor instance self-flagged a stale model pin: a deployed auditor file still named its model (Fable 5) as current after that model was withdrawn earlier than its announced 2026-06-23 window. Rather than patch the instance, fix the *source* so future auditor roles don't carry the brittle form — "suggest using the preferred model if it's accessible, otherwise defer automatically to the most current version of the leading model with the most appropriate settings."

### Changed
- **The model-diversity deployment recommendation is now a self-healing standing rule** (`README.md` §"Deployment recommendation: model diversity"; `adoption-guide.md` v0.3.3 note + checklist; `template-auditor-role-file.md` v0.3.6 skeleton). Prior guidance told adopters to record a *dated hard-pin* of the builder/auditor model pairing — which went stale in running instances when a preferred model was withdrawn before its announced window, leaving auditor files naming a model no longer available. The recommendation now reads: state a *preferred* pairing and use it whenever accessible; when a preferred model is unavailable, **auto-defer to the most current leading model available at the most appropriate settings** rather than carrying a stale name; re-pin only when the model or effort/method assignment materially changes.

### Added
- **The same-model decorrelation mechanism: effort/method asymmetry.** When model diversity is temporarily impossible (only one model available), decorrelation rests on the separate-context / separate-file / asymmetric-corpus separation **plus an effort/method asymmetry** — the auditor runs at a higher effort/method tier than the builder (elevated thinking tier; multi-agent adversarial orchestration where available), holding the capability floor by *method* rather than model tier, never by downgrading the builder. Codified from a running pair that landed on a single model after an early model withdrawal and held decorrelation this way.
- **`template-auditor-role-file.md` v0.3.6** — the skeleton's Operational Constraints gains a model/method-pairing clause in the self-healing form, so every newly generated auditor role starts with the auto-deferring rule rather than a brittle dated name. `follows_pattern:` example unchanged: model/method pairing is a Recommended deployment rec, not a conformance criterion or other conformed surface, so advancing the pin would be the re-pin treadmill this kit retired (v0.4.2 pin semantics).

### Retained unchanged
- No conformance-criteria or message-protocol changes; all v0.5.x channels and existing instances remain conformant (C-22/C-23 unaffected). Existing auditor instances carrying a dated pin are not non-conformant — they adopt the self-healing phrasing at their next role-file revision.

## [0.5.2] — 2026-06-15

### Added
- **`comms-spec.md` v0.5.2 — §Activation backstop: the per-turn unread hook.** A `UserPromptSubmit` hook (reference helper `.comms/bin/comms-unread-banner`) injects the **cursor-delta** unread count (never inbox file-count — immutable mail never leaves the inbox) into the agent's context **every turn**, so an agent notices waiting mail at the next turn boundary **regardless of whether the activation loop is running** — the structural backstop for "a signed mechanism not running protects nothing" (a running auditor with `comms-wait` down missed mail ~20 min and misdiagnosed who was behind). Fires at the turn boundary (mid-turn ingest is the honest structural limit); composes with `heartbeat wait` (between-turn re-invoke) as belt + suspenders. Reference helper is slug-agnostic (zero-config, all inboxes) or slug-scoped per session (settings-isolated); read-only, exit-0-advisory, silent-when-clear. The per-turn unread *surface* moves from §"does not cover" to a provision; the **wake-on-arrival hook** (waking a *live* session mid-task) remains the harness boundary. No message-protocol or criteria changes; v0.5.x channels remain conformant.

## [0.5.1] — 2026-06-15

### Changed
- **`comms-spec.md` v0.5.1 — §Activation: poll interval COLLAPSED to a single uniform low rate** (default ~5s, hard-max-clamped), replacing v0.5.0's "bounded-adaptive tight/rest" language. Per the running auditor's triangulated refutation of the prior adaptive design: the split's receiver-side tight-entry was never built (the in-flight marker had no runtime writer — only the selftest sets it), so the only live effect was the idle rate; polling is ~free (pure bash, no model tokens), so a uniform low rate dominates on cost + simplicity. **Surgical** — only the cadence branch was removed from the reference `cb_wait_interval`; the in-flight marker is retained for its other two consumers (the teardown-refusal guard + the health-line observability), enumerated and verified before the delete. Reference selftest gains an `interval-uniform-ignores-marker` regression guard; reproduced GREEN by the auditor. No message-protocol or criteria changes; v0.5.0 channels remain conformant.

## [0.5.0] — 2026-06-15

### Origin
Owner directive: make the builder↔auditor comms reliable "from the start, every single time." The reliability layer was built and proven on a running pair (brainstorm→design→M1 CONFORMANT→M2 reliability-signed→cross-death closure), each gate independently reproduced by the running auditor instance, and deployed live before the spec text was written — the proven-then-specified order this kit prefers. An **evolution of one layer** of `comms-spec.md`, not a rework: the message protocol is unchanged.

### Added
- **`comms-spec.md` v0.5.0 — §Reliability** (evolves the v0.4.0 watch economy). Liveness by heartbeat **age** (a passive stamp, never process-presence — no "who watches the watcher"); the reader cursor gains an authoritative processed-**set** (`cursors/<slug>.done`) so *unread* is an order-independent set difference advancing on **action, not receipt** (no message lost — level-triggered). A unified self-locating command surface: `up` (idempotent self-verifying singleton/detached bring-up), `wait` (encoded bounded-adaptive in-session activation), `down` (by-PID teardown + stand-down marker), `doctor`/`status` (marker-first LIVE/DORMANT/DEGRADED), **`wake`** (stateless out-of-process cross-death closure), **`selftest`** (the deterministic failure-injection suite — reliability as a *falsifiable* property, identical-GREEN over N runs). Premises 4-5 (liveness-by-age; no-message-lost) and the command-surface contract added. The cross-session wake **trigger** and its **project list** are deployment-local and explicitly NOT part of the kit (private projects are never named in a shared spec).
- **`conformance-criteria.md` v0.3.2 — Criterion C-23 (Recommended)**: comms reliability provisions adopted and proven (command surface present; `heartbeat selftest` reproducibly identical-GREEN; liveness-by-age; single honest heartbeat writer). Recommended because the channel (C-22) functions without the reliability layer; the layer hardens unattended / at-scale pairs.
- **`README.md`** — front-door synced to v0.5.0: comms-spec file-table row (§Reliability + command surface), criteria count (22 → 23, five → six Recommended), Status section, Roadmap entry, and a "Liveness by age" glossary term.
- **`template-auditor-role-file.md` v0.3.5 + `adoption-guide.md` v0.3.2** — the adopter-facing comms-duty clauses synced to §Reliability: `heartbeat up`/`wait` + the `.done` cursor replace watch-arming; commit-signal hook installation + the presence check removed (superseded). Skeleton/Step-6 `follows_pattern:` example advanced to v0.5.0 (comms duties are a conformed surface that moved). So a new adopter copying the skeleton builds the v0.5.0 model, not the retired one.

### Superseded
- **Commit-signal hooks** (comms-spec v0.4.0) — liveness-by-age plus the level-triggered durable cursor make a commit file-event unnecessary; retained-optional, no longer a spec provision.
- **The scheduled wake companion** (comms-spec v0.4.0) — generalized into the liveness-aware `heartbeat wake` (fires only on a true stall, not on every unprocessed message).

### Retained unchanged
- The message protocol: message-per-file inboxes, immutability, reply-link state, thread closure, corpus boundary, the finding schema, housekeeping. Backward-compatible — v0.4.0 channels remain conformant (C-22), gaining C-23 only where the reliability layer is adopted.

## [0.4.2] — 2026-06-11

### Changed
- **`follows_pattern:` pin semantics: the pin declares the conformed version** (template v0.3.4 guidance; C-1 check refined at conformance-criteria v0.3.1). Owner-ratified resolution of the design fork the RoleSmith Auditor's P3 flag made explicit: the v0.4.0 re-pin wave itself shipped as release v0.4.1, staling both instance pins at the moment of their repair — under tracks-latest semantics, the treadmill's repair *was* the treadmill. Under conformed-version semantics the pin is stale only when a later release changes a surface the instance must conform to; content-irrelevant releases (this one included) stale nothing. Existing instance pins at v0.4.0 are conformant with no edits required.

## [0.4.1] — 2026-06-11

### Fixed
- **Template skeleton `follows_pattern:` example advanced to v0.4.0** (template v0.3.3). It had regressed to two releases stale (v0.3.0) — repeating the exact history the v0.3.0 entry documents. Surfaced when a running instance (Deep Thought Auditor) flagged the identical stale pin in its own file; disposed by enumeration across all three pin surfaces (both running auditor instances re-pinned in their own provenance, this skeleton here). The deliberate-pin maintenance recurring as designed, second documented cycle.

## [0.4.0] — 2026-06-11

### Origin
Owner directive: make inter-role monitoring token-efficient — cost scaling with events, not time — without the owner re-spinning monitors. The design went through a full adversarial review by a running auditor instance over the mail system itself before any spec text was written (`2026-06-11-design-review-findings-watch-economy`, six findings); the two structural refutations reshaped the design.

### Added
- **Watch-economy section in `comms-spec.md` (v0.4.0).** Reader-owned cursors (`.comms/cursors/<role-slug>`) — the reviewer's inversion of the proposed writer-maintained sentinel: the head is derivable from the directory, atomic with filing; the cursor is the reader state that died with sessions; single-author by construction, no co-edited file; a writer-maintained freshness signal is rejected as a claim batched with the filing act and unconditioned on it. Watch-lifetime rule: session-long by default; thread-scoped watches permitted **only as a package with the scheduled wake companion** (archive evidence showed auto-disarm alone reinstates the owner as scheduler and orphans quiet-period mail for low-cadence pairs). Commit-signal hooks (`post-commit` + `post-merge` touching `.comms/.commit-signal`) with honest event scope (fetch-plus-reset fires neither), setup-duty installation, session-start presence check (hook absence becomes a conformance fact, not config faith), and the latency-only claim: delivery still verifies at the ref; signal absence is evidence of nothing.
- **Rule 7: the id's date is a claim** — verify against the clock at filing; misdated ids correct by follow-up message, never rename. Precedent: two independent misdated-id instances in one day, one per party.
- **Template skeleton (v0.3.2)** comms clauses aligned (watch-lifetime reference, cursor update, hook installation + presence check in the duties). **Adoption guide** Step 9 wording aligned and made version-free. No criteria changes; C-22's duty enumeration covers the extended duties by reference to the spec.

## [0.3.1] — 2026-06-11

### Added
- **Trigger-source provision in the `artifact-verdict-only` posture** (`template-auditor-role-file.md` v0.3.1). Verdicts may fire on request or — when the owner directs it — on artifact events (work-surface file watches; a file event is an audit trigger, nothing more). Event triggering changes *when* audits fire, never *how* they read: the cold-context discipline holds per verdict regardless of trigger source, so the posture value is unchanged by event triggering. Comms participation is explicitly not monitoring. Origin: an owner-directed artifact watch landed on a running instance the same day; the instance operated under owner authority while logging the uncodified extension as a divergence record — the gap closed instance-first (both running instances), template-second (here). No criteria changes.

## [0.3.0] — 2026-06-11

### Origin
Owner direction to standardize the builder↔auditor mail system that ran operationally in two projects (the Aire RoleSmith pair and the Sketch Builder/Auditor lanes) before being specified here — the reverse of the usual order, and the healthier one: the spec describes a system with an operational day behind it, including a full finding→remediation→verification→promotion lifecycle carried entirely over mail.

### Added
- **`comms-spec.md` (new sibling spec).** Message-per-file inboxes; immutable messages with reply-link state and thread closure; session-start read-then-arm discipline (persistent inbox watches — the harness wakes the role, removing mid-session human relaying); the corpus boundary (mail is not corpus; C-16 undisturbed); pin-free body practice; multi-writer filename variant and incremental-migration seam for multi-role projects. Design premise stated up front: a role cannot wake itself, but it can arm things that wake it.
- **Auditor-owned setup with the activation-directive bootstrap.** Standing the channel up is the auditor's first-boot duty: create the tree, file the activation directive into the builder's inbox — the channel's first message is the builder's operating manual. The builder's session-entry configuration (CLAUDE.md or equivalent) carries only a one-line bootstrap pointer, added during the routing step — resolving the chicken-and-egg (mail cannot teach a builder to read mail) while preserving Step 7's single-required-edit property for the audited role file. Embedding comms discipline in the audited role file is optional hardening.
- **Auditor-owned housekeeping, self-triggering.** Threshold sweep (>20 files → relocate answered mail except the 10 newest to archive), relocation-only (immutability survives archiving), unanswered mail never archives, sweep notices as the log, move-safe id resolution. The duty rides the session-start read; no human prompts it.
- **Adoption guide Step 9** (comms wiring; Surface renumbered to Step 10; checklist extended). **Template skeleton** gains the comms discipline and setup/housekeeping duty clauses plus the verdict-destination update. **Criterion C-22 (Recommended)**: comms channel established per comms-spec — Recommended because the pattern functions with owner-relayed verdicts; the channel removes the human relay.
- **Pattern spec**: `.comms/` row added to the file-relationship topology; **corpus spec**: mail-is-not-corpus stated as access-discipline rule 6.

### Changed
- All sibling specs aligned at 0.3.0 lockstep. `follows_pattern:` example pins in the template skeleton and guide Step 6 updated to v0.3.0 (they had sat at v0.2.3 for three releases — the deliberate-pin exception is recurring maintenance by design, and it recurred).

## [0.2.6] — 2026-06-11

### Origin
Owner decision following the separate-file architecture's first operational day. The deciding evidence: a separate-file auditor (RoleSmith Auditor) caught its own author reproducing, in a second auditor file, the exact defect class the author had fixed in the first file hours earlier — the correlated-blind-spot failure mode that same-file architecture structurally cannot catch, observed and remediated in production. Supporting evidence from the same day: the detection/maintenance boundary held when the auditor flagged a defect in its own role file without editing it; refutation-framed verdicts produced honest outcomes including a refusal to assert an unverified remediation; the spec→instance→finding→spec loop closed within the day (three v0.2.5 changes trace to instance findings).

### Removed (BREAKING for v0.1.x deployments)
- **The same-file (v0.1.x) architecture is retired entirely.** The `archive/v0.1.x-same-file-architecture` branch is deleted; v0.1.x is no longer a supported pin point. Content survives in git history for archaeology, not adoption. All "adopters can stay on v0.1.x" language scrubbed from the pattern spec, README, and adoption guide. The adoption guide's §"Migration from v0.1.x" is RETAINED as the supported path for remaining same-file deployments (their extraction source is their own role files, not this repository).
- Retirement is a support-status change, not a stability claim: the v1.0.0 gate (two independent projects, three months production) is unchanged.

### Changed
- **Pin-free references convention.** Frontmatter `references:` fields across all sibling specs now name files without versions; current versions live in each file's frontmatter and this CHANGELOG. This applies the drift catalog's own intervention to the repository that catalogs it: version-pinned cross-references produced four maintenance instances across three releases (E2, R2a, R2b, plus the v0.2.5 refresh); the pin-free form removes the drift surface rather than correcting its instances. Exception kept deliberately: the template skeleton's `follows_pattern:` retains its version pin — an adopter's pattern-version declaration is a C-1 conformance surface, not a sibling cross-reference.
- All sibling specs aligned to version lockstep at 0.2.6.

## [0.2.5] — 2026-06-11

### Origin
First lessons from running instances, all sourced from the RoleSmith Auditor's first operational day (smoke test through first cross-project verdict) and the owner's deployment observations. The pattern's feedback loop is now real: an instance audited another instance, the findings came home, and the generalizable parts land here.

### Added
- **C-21 multi-file rule-source note** (`conformance-criteria.md` v0.2.5). When an auditor's interpretations target rule units in files other than the pinned master (module specs under a master role file), the single `audits_version:` pin can match while the targeted files drift — coverage is conventional, not mechanical. Auditors in that topology SHOULD enumerate targeted files in their staleness clause and treat targeted-file version changes as staleness. Multi-pin map form deferred to v0.3.0. Source: RoleSmith Auditor's verdict on the Deep Thought Auditor (note 2), promoted upstream by owner decision in lieu of a local corpus entry.
- **Availability-window deployment point** (README §Model diversity, point 5). A pinned model with a known access expiry is a staleness with a date: state the window and the planned post-expiry pairing in the clause, converting discovered drift into a scheduled transition. Source: owner's observation that a current pairing's model is billing-limited to a known date neither pin had recorded.
- **Cause-over-output authoring rule for drift catalogs** (`audit-corpus-spec.md` v0.2.5, Category C). Drift-pattern fields SHOULD name the causal mechanism, not only the observed output — "edits update the target line and never re-read its neighbors" produces an intervention (pin-free phrasing); "references go stale" produces only vigilance. Source: instance four of the stale-cross-references class, a version pin that survived three revisions because it sat one line below the edits ("proximity without participation").

### References updated
- `conformance-criteria.md` and `adoption-guide.md` references fields → `audit-corpus-spec.md v0.2.5` (adoption-guide v0.2.5 is references-only).
- No criteria added or changed; v0.2.0-conformant implementations remain conformant under v0.2.5.

## [0.2.4] — 2026-06-11

### Added
- **Model-diversity deployment recommendation** (README new section + adoption guide Step 8 note + checklist item). The separate-file architecture removes shared source text between builder and auditor; it does not remove shared priors — same-model builder/auditor pairs share training-shaped reasoning tendencies that survive every file boundary. Recommendation, in priority order: (1) run the auditor on a **different model** than the builder; (2) **capability floor** — the auditor's model is at least as capable as the builder's, never weaker (a weaker auditor manufactures the capability blind spots the pattern names as unsolved); (3) **elevated thinking tier** for the auditor as a complement on top of model diversity, or as a fallback when only one model is available — tier elevation improves thoroughness within the same lens (mitigates sustained-load attention failures) but does not decorrelate priors. Specific model names are pinned, dated, in the adopting project's auditor file (Operational Constraints), not in this spec — naming models normatively in a domain-agnostic spec is an instance of the stale-reference drift class this project catalogs. Empirical basis: the v0.2.3 errata were caught by re-reviewing v0.2.2's artifacts on a different model than the one that authored them.
- Like routing (Step 8), model selection is invocation-environment configuration invisible to file-level conformance review; no conformance criteria added or changed. v0.2.0-conformant implementations remain conformant under v0.2.4.

## [0.2.3] — 2026-06-11

### Origin
All items in this release originate from an ASA fresh-pass self-review of v0.2.2. The headline observation: **the stale-reference defect class that v0.2.2 fixed (E1–E3) recurred inside v0.2.2 itself** — two reference fields the release touched around but did not update, plus counts and a contradiction the new Step 8 invalidated. Five same-shape instances now observed (E1, E2, E3, R2×2), which crosses the corpus spec's own three-instance threshold for naming a Category C drift pattern; adopting corpora should seed "stale cross-references after multi-file revision" as their first drift-catalog entry. v0.2.0-conformant implementations remain conformant under v0.2.3; no Required criteria added or changed.

### Fixed (errata)
- **R1 — README contradicted Step 8.** The README stated the pattern is "operational the moment the auditor file lands and the corpus tree exists" — directly contradicting v0.2.2's Step 8, which established that an adoption with unwired routing sits inert. README also still said "Seven-step walkthrough" / "Seven steps" (the guide has nine), its adoption summary had no routing step, and its conformance-criteria line said twenty criteria / three Recommended. All corrected; the adoption summary now carries the routing condition explicitly.
- **R2 — E2's defect class recurred in two files.** `adoption-guide.md` frontmatter `references:` still pinned all three siblings at v0.2.0; `template-auditor-role-file.md`'s outer frontmatter `references:` still pinned `auditor-pattern-spec.md v0.2.0`. Both corrected to current versions.
- **R3 — Stale criterion count in Step 8.** "None of the nineteen conformance criteria" was off by one at landing (C-20 shipped in the same release). Replaced with count-free phrasing, since embedded counts are themselves an instance of the drift class this release documents.
- **R4 — C-20 filed out of order and orphaned from the review procedure.** C-20 physically preceded C-19 in `conformance-criteria.md`, and the review procedure's closing line still read "Recommended criteria (C-18, C-19)." Numerical order restored; closing line now reads C-18 through C-21.

### Added
- **`audits_version:` interpretation pinning + Criterion C-21 (Recommended).** The `audits:` pointer is path-level only: when the audited role bumps its version and renumbers or revises its checks, the auditor's per-criterion interpretations silently drift out of correspondence, detectable only when a full C-10 review happens to run. The auditor now pins the audited-role version its interpretations target (`audits_version:` frontmatter) and compares it against the audited role's current version at session start, flagging staleness before any verdict. Template skeleton, Operational Constraints, annotated guidance, adoption guide Step 6, and checklist updated. Follows the C-20 precedent of landing Recommended criteria in a patch release.
- **Structured finding schema + Category A mapping.** The template's Outputs findings-list entry expanded into a named-field schema (`id`, `check`, `proposition`, `derived`, `observed`, `result`, `citation`, `corpus_refs`) that maps field-for-field onto Category A corpus entry frontmatter per the corpus spec's new §"From finding to entry." Closes the previously undefined gap between "auditor MAY draft a candidate Category A entry" and the entry schema — promotion is now mechanical, and future active-learning Layer 1 tooling gets a parseable verdict-to-corpus trail. The schema also disambiguates the `result` vocabulary: the result names the fate of the conformance proposition, not the refutation attempt.
- **Inconclusive-density verdict guidance.** Check-level `inconclusive` results had no implementation-level grading consequence. New convention: a verdict resting on more than roughly a quarter inconclusive checks SHOULD grade no higher than CONFORMANT WITH NOTES, with notes enumerating each inconclusive check and the evidence that would resolve it.

### References updated
- `adoption-guide.md` frontmatter `references:` → `auditor-pattern-spec.md v0.2.2, template-auditor-role-file.md v0.2.3, audit-corpus-spec.md v0.2.3`
- `template-auditor-role-file.md` frontmatter `references:` → `auditor-pattern-spec.md v0.2.2`
- `conformance-criteria.md` frontmatter `references:` → `auditor-pattern-spec.md v0.2.2, template-auditor-role-file.md v0.2.3, audit-corpus-spec.md v0.2.3`
- `template-auditor-role-file.md` skeleton `follows_pattern:` → `Project Prestidigitonium v0.2.3`
- `auditor-pattern-spec.md` unchanged at v0.2.2 (no content changes this release).

## [0.2.2] — 2026-06-07

### Origin
All items in this release originate from Sketch Main Auditor's review of v0.2.0 at commit `d436e33`. Five errata (E1–E5) plus two design questions (Q1, Q2) — all incorporated. v0.2.0 implementations remain conformant under v0.2.2; no Required conformance criteria added or invalidated.

### Fixed (errata)
- **E1 — Wrong criterion reference in adoption-guide.md failure-modes list.** Cited C-14 (relational primitives) where C-16 (load-list discipline) was meant. Corrected.
- **E2 — Stale frontmatter `references:` field in audit-corpus-spec.md.** Still pinned at `auditor-pattern-spec.md v0.1.0` after the v0.2.0 body update. Now points at v0.2.2.
- **E3 — Three stale §Audit-Variant references in audit-corpus-spec.md body.** Disk-layout diagram, status-field explanation, and post-mortem incorporation rule each carried v0.1.x language that the v0.2.0 pass missed. Scrubbed; all three now correctly reference the auditor role file's audit interpretations / cross-rule obligations.
- **E4 — Internal contradiction between audit-corpus-spec.md and conformance-criteria.md C-15.** Spec said Category D directory contains "no files"; C-15 said it contains an explanatory README. Reconciled with the C-15 version (README present); empty directory wouldn't survive git anyway.
- **E5 — Semver-language contradiction in auditor-pattern-spec.md.** Defined Major as `x+1.0.0` then labeled 0.1.1→0.2.0 a "major architectural revision" / "major version bump." Defensible under 0.x convention but the labels contradicted the scheme. Corrected to "breaking revision under the 0.x convention."

### Added (design questions)
- **Q1 — Step 8 (and Step 9 split) in adoption-guide.md.** New "Wire project-harness routing so auditor instances actually bind to the auditor file" step closes the gap where a fully conformant adoption could sit inert because the project harness keeps routing auditor invocations to the audited role's file. None of the nineteen conformance criteria catch this failure because routing is project-environment configuration. Step 8 names the obligation explicitly, references the three routing strategies (added in v0.2.1), and requires a smoke test. Total step count: nine.
- **Q2 — Cold-context posture clarification + `audit_posture` declaration mechanism + Purpose-skeleton generalization in template-auditor-role-file.md.** The original cold-context clause was correctly scoped to artifact-verdict audits but read as forbidding monitoring functions wholesale, which would contradict standing directives for continuous-monitoring auditors. Three changes resolve the ambiguity: new `audit_posture` frontmatter field with three valid values (`artifact-verdict-only`, `continuous-monitoring`, `hybrid`); new "Audit posture declaration" body section requiring explicit declaration with graceful default to `artifact-verdict-only`; Operational Constraints cold-context clause clarified to scope strictly to the artifact-verdict step (monitoring functions explicitly compatible with `continuous-monitoring` / `hybrid` postures). Purpose skeleton also generalized: previous "Audit role specifications" opening assumed role-file audits; replaced with `<audited role's output type>` slot.
- **Criterion C-20 (Recommended) in conformance-criteria.md.** Verifies `audit_posture` declaration is explicit (frontmatter + body match). Recommended (not Required) because v0.2.2's template specifies a graceful default; explicit declaration prevents downstream ambiguity but absence is operationally tolerable.

### References updated
- `audit-corpus-spec.md` frontmatter `references:` → `auditor-pattern-spec.md v0.2.2`
- `conformance-criteria.md` frontmatter `references:` → `auditor-pattern-spec.md v0.2.2, template-auditor-role-file.md v0.2.2, audit-corpus-spec.md v0.2.2`
- `template-auditor-role-file.md` skeleton frontmatter `follows_pattern:` → `Project Prestidigitonium v0.2.2`

## [0.2.1] — 2026-06-07

### Added
- `auditor-pattern-spec.md` v0.2.1 — Explicit supersession of v0.1.x's "no separate file" architectural language. Quotes the v0.1.x template's note ("The role IS the capability; `audit` is one of two lenses... there is no separate `*-auditor.role.md` file") and the v0.1.x adoption-guide statement that adopters wondering if they needed one "do not." States the reversal under v0.2.0+ so adopters pinned to v0.1.x reading the confident "no" understand the project itself has changed direction.
- `auditor-pattern-spec.md` §"The shift in plain terms" — articulates *why* the architectural inversion is an improvement rather than a reorganization: v0.1.x asymmetry was enforced by load-discipline (reminder, not gate); v0.2.0 asymmetry is enforced by the file boundary itself (structural, not declarative). Framing credited to Sketch Main Auditor's observation.
- `adoption-guide.md` v0.2.1 — New "Project CLAUDE.md routing under separate-file architecture" section documenting three routing strategies (Strategy A: separate CLAUDE.md per role/auditor directory, recommended; Strategy B: single CLAUDE.md with explicit instance dispatch; Strategy C: auditor invocation bypasses CLAUDE.md auto-load). Addresses the salience-contamination risk when a hardcoded CLAUDE.md routes both builder and auditor invocations to the audited role's content.

### Origin
Both additions originated from Sketch Main Auditor's review of the archive branch + main bookmark before v0.2.0 was pushed. Their observations: (1) the v0.1.x confident "no" needed explicit forward supersession in v0.2.0; (2) CLAUDE.md routing under v0.2.0 deserved adoption guidance, not just structural-requirement implication. Both incorporated.

### Conformance
v0.2.1 is editorial relative to v0.2.0 — no integration-requirement changes; no conformance-criteria changes. v0.2.0 implementations remain conformant under v0.2.1 without modification.

## [0.2.0] — 2026-06-07

### Changed (BREAKING — major architectural revision)
- **Architecture inverted from same-file to separate-file.** v0.1.x committed to a `# §Audit-Variant` section in the audited role's file. v0.2.0 inverts to a separate `<audited-role-slug>-auditor.role.md` file with the audited role declared as an Input via reciprocal `audits:` / `audited_by:` frontmatter pointers. Reasoning: same-file preserved domain fluency at the cost of shared source text — the auditor reading the same words could not catch what those words did not say. Mechanism-level mitigations (asymmetric corpus, adversarial frame, independent re-derivation) addressed the symptom; separate-file addresses the structural cause.
- **The four mechanisms survive the inversion** (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation). Their operationalization moves from "section in the audited role file" to "structure of the separate auditor file."

### Removed
- `template-audit-variant-section.md` (v0.1.x). Replaced by `template-auditor-role-file.md`.
- The `# §Audit-Variant` section pattern in audited role files. Audited role files contain no audit lens content under v0.2.0; the auditor lives in its own file. Adopters migrating from v0.1.x extract the section content into the new auditor file per `adoption-guide.md` §"Migration from v0.1.x."

### Added
- `template-auditor-role-file.md` v0.2.0 — Full auditor role file skeleton (frontmatter through Change Control), replacing the v0.1.x section template.
- Conformance criteria C-1 (separate auditor role file exists with correct frontmatter), C-2 (reciprocal `audited_by:` pointer), C-17 (no `# §Audit-Variant` section in audited role file).
- `adoption-guide.md` §"Migration from v0.1.x" — five-step migration path for v0.1.x adopters.
- `auditor-pattern-spec.md` §"File-relationship topology" — declares the audited / auditor / governance / corpus file relationships explicitly.

### Branch policy
- `main` tracks v0.2.0+.
- `archive/v0.1.x-same-file-architecture` preserves v0.1.0 + v0.1.1 verbatim. Adopters who deployed against v0.1.x can pin to this branch.

## [0.1.1] — 2026-06-06

### Added
- `audit-corpus-spec.md` v0.1.1 — Active-learning hooks: documents machine-readable frontmatter surfaces, detection-layer signal types, promotion-layer hooks, evolution-layer caution, and adopter discipline that produces active-learning-ready corpora today. Adds the `informed` field to the entry frontmatter schema as the cross-category linkage signal (Category A → Category C promotion provenance). Forward-compatible patch; v0.1.0 implementations remain conformant.

### Deferred
- Active-learning extension specification (anticipated `active-learning-spec.md`). Authored when accumulated real corpora provide evidence to design against rather than designing in the abstract.

## [0.1.0] — 2026-06-06

### Added
- `auditor-pattern-spec.md` v0.1.0 — canonical specification: encompassment principle; four mechanisms (encompassment scope, asymmetric failure-pattern corpus, adversarial default, independent re-derivation); naming convention (`<Project> <Role> Auditor`); three-level authority precedence; scope-limit acknowledgments; integration requirements; semantic versioning policy.
- `template-audit-variant-section.md` v0.1.0 — copy-pasteable `# §Audit-Variant` skeleton with annotated companion explaining each preamble element, per-rule clause structure, and cross-rule obligation set.
- `audit-corpus-spec.md` v0.1.0 — structure, growth rules, and access discipline for the asymmetric corpus: five entry categories (prior findings, failure post-mortems, drift catalogs, self-referential §Audit-Variant clauses, project-specific materials); default disk layout; entry frontmatter schema; bootstrapping guidance for v0.1.0-empty corpora; anti-pattern catalog.
- `adoption-guide.md` v0.1.0 — seven-step walkthrough for instantiating the pattern in an existing role file; what-changes-for-builder and what-changes-for-auditor sections; recurring adoption failure modes; compact adoption checklist.
- `conformance-criteria.md` v0.1.0 — sixteen conformance criteria (fourteen Required, two Recommended); four-level verdict structure (CONFORMANT / CONFORMANT WITH NOTES / PARTIALLY CONFORMANT / NON-CONFORMANT); three-pass review procedure (structural / preamble / clause).
- `README.md` — front-door overview; glossary of load-bearing terms; rationale for separate repository; adoption summary; roadmap.
- `LICENSE` — Apache 2.0 (carried from initial repository creation).
