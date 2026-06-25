# Comms Bring-Up Directive — correct out of the gate

> **Part of Project Prestidigitonium.** The ordered, idempotent, gate-verified procedure for standing the builder↔auditor comms channel up so it works **as designed on the first try**. This is the operational expansion of `adoption-guide.md` Step 9 (comms wiring), grounded in `comms-spec.md` §Setup / §Layout / §Rules / §Reliability / §Activation backstop and `conformance-criteria.md` C-22 / C-23. It changes **no mechanism and adds no spec requirement** — it *sequences* and *verifies* what `comms-spec.md` already mandates, plus it operationalizes, at the deployment layer, the one capability the spec deliberately leaves to the harness (the wake-on-arrival hook — see §4b / Gate #7).

**Purpose.** A single, ordered, idempotent procedure that any role adopting Project Prestidigitonium follows so the builder↔auditor comms channel works **as designed on the first try**. The recurring failure is not the mechanism — it is the *bring-up*: setup declared "done" on file-existence, never on proof the channel delivers, surfaces, gets read, gets replied to, and advances a cursor.

**Status:** v2.4 — adds the `heartbeat gate`/`doctor` **version-skew conditional** (match the health command to the comms-spec version your `bin/` implements — a pre-armed-listening `bin/` has no `gate`) and reflects **`comms-spec.md` v0.6.2** armed-listening hardening (own-session fail-closed + the coded live-task check). v2.3 added **armed-listening** (the wake-marker) to §4b + the Standing health check (a quietly-died wake is *detectable*; the bring-up gate is a continuous `heartbeat gate`). Derived from `comms-spec.md` (v0.6.0+); pin-free in body — versions live in the spec and the CHANGELOG.

---

## Why this keeps going wrong (root-cause diagnosis, from live failures)

The mechanism is sound; the *bring-up* drifts. Five root causes, each observed first-hand:

1. **Copy-from-instance, not derive-from-spec.** Files cloned from a drifted deployment propagate the drift. *(Dropped corpus-boundary clause; stale spec self-cite; copied cadence from an unrelated deployment.)*
2. **No end-to-end verification gate.** Setup declared "done" because the **files exist** — never because mail was proven to deliver, surface, get read, get replied to, and advance a cursor. Breakage surfaces later as missed mail and chaos.
3. **Shared-checkout scoping traps.** When both legs run at the same cwd (shared `.claude/`), a *slug-scoped* per-turn unread hook covers only ONE party; the other silently gets no per-turn surface of its own mail.
4. **Two-party coordination gaps.** "Who creates the tree / who brings up which watch / in what order" under-specified → orphaned watches and concurrent clobber.
5. **No autonomous-wake — only a per-turn surface plus a long timer.** The per-turn banner (root cause #2's fix) fires only on a *submitted turn*; a heads-down / loop / scheduled agent submits none, so its only wake is whatever timer it set. A long timer alone drops mail into the dead window between fires. *(Live: a loop relying on a long fallback-wake interval with no inbox event-monitor missed consecutive messages that arrived between fires; caught only by owner intervention.)*

**The fix is not new mechanism.** It is: DERIVE every file from `comms-spec.md` (never from an instance); follow ONE ordered two-party sequence; arm a wake that fits the operating mode (per-turn surface for interactive turns, event-wake for heads-down runs); and pass a MANDATORY verification gate — including proof the surface and the autonomous-wake actually FIRE — before any work begins.

---

## Roles in this directive
- **AUDITOR** = the leg that *stands the channel up* (`comms-spec.md` §Setup assigns setup to the auditor).
- **BUILDER** = the audited leg. The builder does **not** create the tree; it adds one bootstrap line and brings up its own watch.

---

## The bring-up sequence

### 0 — Fix BOTH slugs first (before any file exists)
- Convention: builder slug `<project>-<role>`, auditor slug `<project>-<role>-auditor`. Inbox dirs `inbox-<slug>`; heartbeats/cursors keyed by `<slug>`.
- **Both slugs decided together, never guessed later.** A slug the two legs don't agree on = a silently one-way channel.

### 1 — AUDITOR creates the tree, DERIVING every file from `comms-spec.md`
- `.comms/{bin/, inbox-<builder>/, inbox-<auditor>/, cursors/, heartbeats/}`.
- `bin/` = the reference heartbeat implementation as a **self-locating COPY** — then prove it resolves `COMMS` to *this* `.comms` (never another project's tree).
- `comms.conf` registering **BOTH** real slugs **+ the `selftest` fixture**, format `slug|inbox|cadence_s|freshness_s`. Choose cadence/freshness deliberately: **freshness ≥ cadence + slack**; do **not** copy cadence from an unrelated deployment. Cite the comms-spec version derived from, or keep pin-free.
- `.comms/README.md` — the protocol **instantiated for this project from `comms-spec.md`** (not copied from another instance). *Most-skipped file (`comms-spec.md` §Layout + §Setup item 1 require it).*

### 2 — AUDITOR files the activation directive into the BUILDER's inbox
The channel's first message = the builder's operating manual. It MUST state **all five** (`comms-spec.md` §Setup item 2) — checklist:
- [ ] (a) where the builder's inbox is (what it reads),
- [ ] (b) where the builder writes (the counterpart inbox),
- [ ] (c) the immutability / reply-link rules,
- [ ] (d) the session-start *read-then-bring-up* discipline **AND the cursor-advance-on-action rule** (`comms-spec.md` rule 5 — advance `cb_mark_processed <slug> <id>` after *handling* a message, not on receipt),
- [ ] (e) **the corpus boundary** ← the clause dropped most often.

Plus: the bring-up command (`heartbeat up <builder-slug>`), the message schema, and the per-party-own-watch-only rule (step 5).

### 3 — BUILDER's bootstrap pointer (the easy-to-forget cross-file edit)
The **BUILDER (or owner)** adds ONE line to the builder's session-entry config (its `CLAUDE.md`):
> At session start, read `.comms/inbox-<builder-slug>/` newest-first, act on unreplied messages, then `heartbeat up <builder-slug>`.

Without it, the activation directive sits in an inbox nothing opens. The **auditor does NOT edit the builder's config** (single-required-edit property; it's the builder's file). *(Placement note: `adoption-guide` situates this in Step 9 (comms wiring); `comms-spec.md` §Setup item 3 homes the obligation with comms — same one-line edit either way.)*

### 4 — Per-turn unread backstop (the INTERACTIVE surface) — SCOPE IT TO THE CHECKOUT TOPOLOGY, AND MAKE IT LIVE
The `UserPromptSubmit` unread-banner hook (`comms-spec.md` §Activation backstop) surfaces waiting mail every turn even when the activation loop is down. Two things must be right — scope and liveness:

**Scope:**
- **Directory-isolated settings** (each leg its own `.claude/`, adoption-guide Strategy A): each leg wires a **slug-scoped** hook → `comms-unread-banner <own-slug>`.
- **SHARED `.claude/`** (both legs same cwd): wire the **slug-AGNOSTIC** form → `comms-unread-banner` with **no slug**. The no-arg form lists EVERY non-fixture slug, so each session sees all counts — **each session reads/drains only its OWN slug's line.** A slug-scoped hook in a shared checkout covers only one party and leaves the other blind to its own mail.

**Liveness (configured ≠ firing).** Claude Code loads `.claude/settings(.local).json` at **session start**; a hook added mid-session is INERT until the next session start. Therefore **wire the hook before the session that needs it starts, or restart the session after wiring.** Until the banner is live, **manually drain your own unread every turn** (`cb_unread <own-slug>` / read inbox newest-first + advance cursor). Gate check #6 proves the banner actually fires.

### 4b — Autonomous-wake backstop (the HEADS-DOWN surface) — when there is no per-turn surface to fire
Step 4's banner fires on a `UserPromptSubmit` — i.e. only when a turn is *submitted* (a human prompt, a `wait` re-invoke, a session resume). In **autonomous operation** — a `/loop`, a scheduled or heads-down agent, any run without per-turn human prompts — that surface **never fires, because no prompt is submitted.** The banner protects the turns that happen; it does nothing for the turns that never do. An agent in that posture, relying on a long external timer alone, misses mail that lands in the dead window between fires (root cause #5).

The fix is an **event-wake on your OWN inbox**, in addition to any fallback timer:

- **Where the host provides an inbox-event / wake-on-arrival primitive** (a file-watch or monitor that re-invokes the agent on new mail): **arm it on your own inbox** for the duration of the autonomous run, so new mail wakes you within seconds, independent of the timer. *(`comms-spec.md` §"does not cover" leaves this wake-on-arrival hook to the harness **by design** — it is deployment/machine configuration, not a spec mechanism. This step operationalizes it where the host has one.)*
- **Plus the spec's in-session activation** (`heartbeat wait <slug>`, `comms-spec.md` §Activation) where the deployment runs it — level-triggered on the cursor-delta, it re-invokes the agent between turns.
- **Keep a bounded fallback poll** (a periodic wake) as a *safety net*, sized to your latency tolerance — **not** as the primary signal. A long timer *alone* is the failure; a long timer *behind* an event-wake is fine.
- **Never arm the event-wake on the counterpart's inbox** — own-inbox only (same orphan-watch hazard as step 5).

This adds **no spec mechanism.** It operationalizes, at the deployment layer, the wake-on-arrival hook `comms-spec.md` explicitly leaves to the harness, and falls back to the spec's own `heartbeat wait` + the cross-death `wake` ladder where no host primitive exists. Gate #7 proves that whichever mechanism your deployment has **actually fires** — and, where the deployment has only the bounded fallback, makes its latency bound explicit instead of implied.

**Armed-listening — make the event-wake's liveness a FACT, not discipline (`comms-spec.md` v0.6.0 §Armed-listening).** The Ongoing-discipline rule below warns *"an event-wake that quietly died is indistinguishable from no mail"* and leaves re-arming to discipline. Where the deployment adopts the **wake-marker**, that residual closes: the armed wake stamps a tight-freshness, **session-bound** marker every poll, so a quietly-died (or never-armed, or other-session) wake reads **`armed=no`** — distinguishable from a live one. Then:
- **`heartbeat gate <slug>`** is the one-command **continuous** check — **live + armed-listening + drained** → READY / NOT-READY — re-runnable at any boot/turn (not only the setup-time Gate #7 seed-and-wake). **Session-scoped:** run it from the leg's OWN session (a cross-session observer reads `armed=no` by design; cross-leg readiness uses the freshness-floor). **Version-skew:** `heartbeat gate` is a **v0.6.0+** (armed-listening) command — a deployment whose `bin/` predates armed-listening has no `gate` and uses `heartbeat doctor` for its health verdict; match the health command to the comms-spec version your `bin/` actually implements, never a higher one (the live skew a v0.5.x deployment hit). Under **v0.6.2**, `armed` additionally requires a **live wake process** (the coded live-task check), so a dead-but-still-fresh wake also reads `armed=no`.
- **The per-turn surface also cues an un-armed boot:** the `UserPromptSubmit` banner (Step 4) surfaces `⚠ boot-incomplete: your wake is NOT armed` for the OWN slug until armed — so "forgot to arm" is caught every turn, by mechanism, not memory.
- **SessionStart convenience (where the host fires it):** a SessionStart hook MAY auto-bring-up the watch + cue the arm, removing even the "run `comms-up`" step. Treat its firing as **verified-per-deployment** (SessionStart fire-coverage across boot types is a deployment fact, not a guarantee); the per-turn surface is the proven floor regardless.

### 5 — EACH leg brings up ONLY its own watch
- Auditor: `heartbeat up <auditor-slug>`. Builder: `heartbeat up <builder-slug>`.
- **Never bring up the counterpart's slug** — it creates an ORPHAN watch whose heartbeat is written by the wrong process, so liveness reads true while the real owner is absent.
- The builder does **not** re-create the tree (the auditor already did) — prevents concurrent clobber.

### Ongoing discipline (after the gate is green — not a one-time setup step)
- **Advance your reader cursor on ACTION** — `cb_mark_processed <own-slug> <id>` after *handling* each message (not on receipt). A stale cursor (advanced for a few messages then left behind) is the live failure mode that drops mail mid-engagement; Gate #4 is setup-time only and does **not** catch it.
- **Until the banner is live**, manually drain your own unread every turn.
- **In autonomous runs**, keep the event-wake (§4b) armed for the whole run and re-arm it after any context reset that tears it down (a `/compact`, a session resume, a host restart). An event-wake that quietly died is indistinguishable from "no mail."
- **Run the Standing health check** (below) at phase boundaries.

---

## MANDATORY VERIFICATION GATE — do NOT declare setup done until ALL pass
*The centerpiece. The recurring failure is declaring "done" on file-existence. Prove the channel instead.*

1. **Liveness:** `heartbeat doctor <slug>` for **both** slugs → verdict **LIVE**, each watch pid belonging to the RIGHT party.
2. **Reliability proof:** `heartbeat selftest` → identical-GREEN, **FAIL=0**.
3. **Isolation:** `bin/` self-locates to THIS `.comms`; `comms.conf` + inbox dirs correctly namespaced; no path or slug reaches another project's channel.
4. **Drain + cursor (setup-time):** after each leg reads its inbox, `heartbeat doctor <slug>` → **unread 0**, and the inbox-vs-`.done` diff is **empty** (cursor advanced on action). *(Setup-time only — see the Ongoing-discipline cursor rule for during-engagement.)*
5. **END-TO-END HANDSHAKE (delivery proof):** the auditor files a message into the builder's inbox requesting a `re:` ack; the builder reads it, replies `re:` into the auditor's inbox, advances its cursor; the auditor confirms the reply landed. This single round-trip exercises **both inboxes** (auditor→builder delivery + builder→auditor reply), proving bidirectional delivery. *(To prove each initiation direction independently, run the handshake from both sides.)*
6. **PER-TURN SURFACE FIRES (the interactive anti-miss proof — DO NOT SKIP):** delivery ≠ the agent being *surfaced* the mail. Seed a test unread into **each** slug's inbox, trigger a `UserPromptSubmit` in **each** session, and confirm the banner injects **that session's own** unread line. A configured-but-inert banner (mis-scoped, syntax-broken, or wired-mid-session-and-not-yet-live) passes #1–#5 while the per-turn surface is dead — the exact failure that drops mail to a heads-down agent. **After confirming the banner fires, clear the seed** — `cb_mark_processed <slug> <seed-id>` — so the verification artifact does not linger as a spurious real unread. *(Live precedent: a channel passed every other check yet the builder missed 4 consecutive messages because the shared banner was scoped to the auditor only; caught only by owner intervention.)*
7. **AUTONOMOUS-WAKE FIRES (the heads-down anti-miss proof — DO NOT SKIP for any agent that runs without per-turn prompts):** Gate #6 proves the *interactive* surface; this proves the *autonomous* one. With the agent in (or simulating) heads-down/loop posture — no human prompt incoming — seed a test unread into its **own** inbox and confirm the **event-wake actually re-invokes the agent within bounded latency** (the host inbox-monitor fires, or `heartbeat wait` exits on the cursor-delta) — **not** "eventually, at the next long fallback-timer fire." A deployment whose only wake is a long external timer passes #1–#6 while a heads-down agent silently accrues missed-mail latency (root cause #5). If the deployment genuinely has only the bounded fallback (no host event primitive and no `heartbeat wait` running), **record that latency bound explicitly** — silence reads as "instant" when it is not. **After confirming, clear the seed** (`cb_mark_processed <slug> <seed-id>`), same as Gate #6. *(Applies only to deployments that run agents autonomously; a purely interactive adoption — every turn human-driven — is covered by Gate #6 and may mark #7 **N/A** with that justification recorded.)*

Until all **SEVEN** pass (or #7 is justifiably N/A), the channel is **not** set up — no matter how complete the files look. Work does not start before the gate is green.

---

## Failure-mode → prevention map (empirical)

| Observed first-time failure | Prevented by |
|---|---|
| `.comms/README.md` missing | Step 1 (README instantiated from spec) |
| Activation directive dropped the corpus boundary | Step 2 five-content checklist (esp. (e)) |
| Builder bootstrap pointer missing → directive never read | Step 3 (explicit owner; exact line) + Gate #5 |
| Per-turn hook mis-scoped in a shared checkout | Step 4 scope (slug-agnostic) + **Gate #6 (surface fires)** |
| Per-turn hook wired but inert until restart | Step 4 liveness (wire-before-start/restart) + **Gate #6** |
| Heads-down / loop agent misses mail (only a long timer, no event-wake) | **Step 4b (autonomous-wake) + Gate #7 (autonomous-wake fires)** |
| Event-wake torn down by a context reset (`/compact`, resume) and not re-armed | Ongoing-discipline re-arm rule + Gate #7 + **the wake-marker makes a died/un-armed wake DETECTABLE (`heartbeat gate` → `armed=no`), not discipline-blind (`comms-spec.md` v0.6.0)** |
| Orphan watch on the counterpart's slug | Step 5 (own-watch-only) + Gate #1 (right pid) |
| Reader-cursor never advanced / goes stale mid-engagement | Step 2(d) + **Ongoing-discipline cursor rule** + Gate #4 (setup-time) + Standing health check (ongoing) |
| Stale version self-cite in `comms.conf` | Step 1 (derive from spec; pin-free or cite derived version) |
| Cadence copied from an unrelated deployment | Step 1 (freshness ≥ cadence + slack, chosen) |
| Slug mismatch → one-way channel | Step 0 (both slugs fixed + registered together) |

---

## Standing health check (run anytime, both legs)
```
heartbeat gate <slug>                                     # v0.6.0+ armed-listening: READY = live + armed + drained (own session). Pre-armed-listening bin/: use `heartbeat doctor <slug>` (no `gate` there)
heartbeat doctor <slug>                                   # want LIVE + unread→0
diff <(ls .comms/inbox-<slug>/|sed 's/.md$//'|sort) \
     <(sort .comms/cursors/<slug>.done)                   # empty = caught up
cb_unread <slug>                                          # your own unread (banner-independent)
```
In an autonomous run, also confirm the event-wake (§4b) is still armed (re-run Gate #7's seed-and-wake if a context reset may have torn it down).

## Alignment
This directive operationalizes `adoption-guide.md` Step 9 (comms) and brings to comms the same prove-it-works rigor Step 8 already requires for routing (its "smoke test"). It changes no mechanism and adds no spec requirement — it sequences and **verifies** what `comms-spec.md` §Setup already mandates, including the §Activation backstop per-turn surface (Gate #6) that file-level conformance criteria do not catch, and the harness-layer wake-on-arrival hook that `comms-spec.md` §"does not cover" leaves to the deployment (Step 4b / Gate #7).

## Changelog
- **v2.4** — the **`heartbeat gate` / `doctor` version-skew conditional**: `heartbeat gate` is a v0.6.0+ armed-listening command, so a deployment whose `bin/` predates armed-listening (e.g. a v0.5.x collaborator) has **no `gate`** and uses `heartbeat doctor` for its health verdict. Match the health command to the comms-spec version the `bin/` implements, never a higher one — stated by pointing to the adopted version, not by re-enumerating the surface (C-23). *(From a live skew: a v0.5.x DT deployment was directed at a `gate` its `bin/` did not have.)* Also reflects **`comms-spec.md` v0.6.2** armed-listening hardening — `armed` now additionally requires a live wake process (the coded live-task check), so the §4b `gate` distinguishes a dead-but-fresh wake too. **No new gate**; Gate #7's autonomous-wake proof is unchanged.
- **v2.3** — **armed-listening** (`comms-spec.md` v0.6.0): the event-wake stamps a session-bound wake-marker, so a quietly-died wake reads `armed=no` — closing §4b's *"an event-wake that quietly died is indistinguishable from no mail"* residual (now **detectable**, no longer discipline-only). Adds **`heartbeat gate <slug>`** (continuous live + armed + drained, session-scoped) to §4b + the Standing health check, the per-turn **boot-incomplete cue** (the `UserPromptSubmit` banner surfaces an un-armed own-slug every turn), and a **SessionStart auto-bring-up** convenience (firing verified-per-deployment, floored by the per-turn surface). Adds **no new gate** — the existing Gate #7 setup-proof stands; `heartbeat gate` is its re-runnable continuous form. Built + `selftest`-proven (43 checks incl. the production enacting path) + decorrelated-auditor-verified before landing (proven-then-written).
- **v2.2** — autonomous-wake provisions: new **root cause #5** (heads-down/loop agents have no per-turn surface; a long timer alone drops mail in its dead window), new **Step 4b** (arm an event-wake on your own inbox — the host wake-on-arrival primitive where it exists, `heartbeat wait` where the deployment runs it, a bounded fallback poll behind both), new **Gate #7** (autonomous-wake fires — prove the event-wake re-invokes a heads-down agent within bounded latency, not at the next long-timer fire; N/A for purely-interactive adoptions), an Ongoing-discipline **re-arm-after-context-reset** rule, two failure-map rows, and the gate count six→**seven**. Operationalizes — at the deployment layer — the wake-on-arrival hook `comms-spec.md` §"does not cover" leaves to the harness; adds no spec mechanism. From a live incident: a loop on a long fallback-wake interval with no inbox event-monitor missed mail in the dead window; fixed by arming a persistent inbox monitor (event-wake) alongside the timer.
- **v2.1** — Gate #6 also clears the seeded test unread (`cb_mark_processed <slug> <seed-id>`) after confirming the banner fires, so the verification artifact does not linger as a spurious real unread.
- **v2** — folded a decorrelated verification: **Gate #6** (per-turn surface fires — the anti-miss proof the v1 gate omitted); Step 4 **liveness** (configured≠firing: wire-before-start/restart + manual-drain-until-live); explicit **cursor-advance-on-action** (Step 2(d) + Ongoing-discipline); shared-checkout "read only your own line"; bootstrap-pointer placement note; Gate #5 bidirectionality clarified.
- **v1** — initial directive (derive-from-spec, ordered sequence, 5-check gate).
