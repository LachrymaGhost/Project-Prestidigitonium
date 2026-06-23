---
name: comms-spec
version: 0.6.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [comms, mail, audit, governance, asymmetry, reliability, liveness]
references: auditor-pattern-spec.md, audit-corpus-spec.md, template-auditor-role-file.md
---

# Comms Specification — Builder ↔ Auditor Mail

## Purpose

This specification defines the **message-per-file inbox mail system** between an audited role (builder) and its auditor: how verdicts, audit requests, flags, and notices travel between the two without a human relaying them, how the channel stays cheap to cold-start, how it maintains itself without prompting, and how it stays **reliable** — live, activating, and recoverable across crashes — without a long-lived daemon anyone has to supervise.

The design premises:

1. **A role cannot wake itself, but it can arm things that wake it.** Duties attach to the step that always runs (session start); the session-start step brings the watch up; the harness does all subsequent waking. The only recurring human act is one spin-up per session.
2. **Context burn is bounded by unanswered mail, not by history.** Answered messages are never re-loaded. There is no monolithic mail file to scan at cold start.
3. **The system bootstraps through its own first message.** The auditor stands the channel up and files the activation directive into the builder's inbox — the first message is the operating manual.
4. **Liveness is a passive fact, not a probe.** A watch's freshness is judged by the **age** of a heartbeat it writes, never by process-presence — so there is no "who watches the watcher" regress and no instrument to fool. *(The load-bearing reliability premise.)*
5. **No message is ever lost; only latency varies.** Unread is a set difference against a durable, reader-owned processed-set that advances on **action, not receipt** — so a missed or late wake costs timeliness, never a message.

## Layout

```
.comms/
├── README.md              — the protocol, instantiated for the project (access rules + housekeeping)
├── comms.conf             — role registry: slug|inbox|cadence_s|freshness_s (§Reliability)
├── bin/                   — the reference heartbeat implementation (role- AND project-agnostic; self-locating)
├── inbox-<builder-slug>/  — read by the builder; written by the auditor and the owner
├── inbox-<auditor-slug>/  — read by the auditor; written by the builder and the owner
├── heartbeats/<role-slug>    — liveness-by-age (§Reliability)
└── cursors/<role-slug>       — reader-owned high-water (display/compat)
    cursors/<role-slug>.done  — reader-owned processed-SET; authoritative for unread (§Reliability)
```

One message per file. Filename: `YYYY-MM-DD-<slug>.md` for two-writer inboxes; `YYYY-MM-DD-<from>-<slug>.md` when an inbox has three or more writers (multi-role projects).

## Message format

```yaml
---
id: <filename basename>
from: <role-slug or owner>
to: <role-slug>
type: audit-request | audit-verdict | reply | notice
re: <id>                        # optional — the message this answers
artifacts: [<paths>]            # files under discussion
---

<body>
```

`audit-verdict` bodies carry the finding schema from `template-auditor-role-file.md` §Outputs, so verdicts map mechanically onto Category A corpus entries (`audit-corpus-spec.md` §"From finding to entry").

**Pin-free body practice:** messages name artifacts and let artifacts carry their own versions. A version quoted in an immutable message is a stale pin with a filing date.

## Rules

1. **Never write into your own inbox** (the auditor's housekeeping relocation excepted).
2. **One author per file**, named in frontmatter. No file is ever co-edited.
3. **Messages are immutable once filed.** Corrections, retractions, and follow-ups are new messages linking the original via `re:`. Nothing is edited or deleted.
4. **Reply-linking is the state.** A message is answered when a message with `re: <its id>` exists in the counterpart inbox. **Thread closure:** a reply to any message in a `re:`-chain answers the chain's root and ancestors — replying to the newest message in a thread closes the thread.
5. **Session-start discipline (both parties):** read your inbox newest-first; act on messages without replies; **then bring your watch up — `heartbeat up <slug>`** (idempotent, singleton, self-verifying; §Reliability). In-session, activation is level-triggered off the durable cursor (`heartbeat wait` exits on unread, which re-invokes the agent). **After acting on a message, advance your reader cursor** (high-water + `.done`) — it advances on **action, not receipt** (§Reliability).
6. **Corpus boundary (load-bearing).** Mail is not corpus. The comms tree lives outside `audit-corpus/`, so the builder reading its own mail never touches auditor-only material (Criterion C-16 is undisturbed). Verdicts MAY cite corpus entries by id; the builder may then read only the cited entries, per the corpus access discipline.
7. **The id's date is a claim.** Verify it against the clock at filing — a date-stamped id filed without reading the clock is assertion-before-verification in its smallest unit (two independent same-day instances, one per party, are this rule's precedent). A misdated id is corrected by a follow-up message linking `re:`, never by rename.

## Reliability — liveness, activation, and recovery without a daemon to supervise

This section supersedes the prior *watch economy* (v0.4.0): it keeps what that bought (a reader cursor; cost that scales with events, not time) and replaces the watch mechanism with **liveness-by-age** plus a unified, self-locating command surface. Each provision below buys a specific thing; the rationales are not interchangeable. None of them changes how verdicts read or how delivery is verified.

### Liveness by heartbeat age
A watch writes `heartbeats/<role-slug>` on a fixed cadence (atomic temp+mv). A reader judges liveness by the **age** of that stamp against a per-role freshness threshold (`comms.conf`). Process-presence is used ONLY for the singleton/teardown identity check (an exact-arg pidfile match), never for liveness — so there is no peer process to mistake for liveness and no supervisor regress. A malformed or empty heartbeat reads **DEAD**, never false-fresh.

### Armed-listening — the wake-marker (watch-liveness ≠ agent-listening)
Liveness-by-age proves the **watch** process is alive — it does **not** prove the agent-waking **wake** is armed, nor that the agent will act on mail. The watch writes the heartbeat regardless of agent state, so a leg can read **LIVE while the agent is compacted, stalled, or dead and mail rots** (the *GREEN ≠ listening* gap), and `heartbeat up` brings the *watch* up without arming the *wake* (the *up ≠ armed* seam). The wake (the harness inbox-monitor / `heartbeat wait`) is the only thing that re-invokes the agent — so it gets its **own** liveness marker, parallel to the heartbeat:

- The armed wake **stamps a wake-marker every poll** (`<run>/<role-slug>.wake`, atomic temp+mv) carrying a UTC stamp **and the session id**. **Armed-listening ≡ the wake-marker is fresh AND session-bound to the checking session.** Freshness is **tight** (a small multiple of the wake poll) so a *dead* wake reads armed for at most that window; a malformed/missing marker reads **NOT-armed**, never false-armed.
- **Session-binding (why a bare timestamp is insufficient):** a marker written by a harness-bound wake and one written by a stale/detached process are byte-identical — freshness alone cannot tell *who* armed it, silently reducing "armed" to discipline. Binding the marker to the session id makes a **stale or other-session** wake fail `armed`. *Honest scope:* this proves *"a fresh wake stamped by THIS session,"* not *"a harness-bound wake"* — a same-session detached wake inherits the id (closed by an agent-step live-task check where the gate runs as an agent action; the bring-up ritual forbids detached launches). Forge-open under same-uid — the on-disk ceiling this spec names elsewhere.
- **This makes a quietly-died wake DETECTABLE** — closing the bring-up directive's named residual ("an event-wake that quietly died is indistinguishable from no mail," left to discipline): a died wake's marker goes stale → `armed=no`.
- **The bring-up gate** (`heartbeat gate <slug>`) asserts **live + armed-listening + drained** and refuses to certify "boot complete" otherwise — replacing *declare-done-on-file-existence* with proof the leg is **actually listening + caught up.** The gate is **session-scoped** (meaningful only run BY that leg's own session; a cross-session observer reads `armed=no` by design — cross-leg readiness uses the freshness-floor, as the cross-death `wake` does).

*Reference-impl hardening folded with this layer:* `cb_mark_processed <slug> <id>` **rejects an empty id** (an omitted id previously appended a blank line and marked nothing — a silent drain no-op); and the display cursor `cursors/<slug>` is labeled **lexical-max-ever / advance-only / display-only** — authoritative unread is always `cb_unread_count` (the `.done` set difference), never the high-water.

### Reader cursor — a processed-SET, not just a head
The head is derivable from the directory (zero added writes). What dies with sessions is the *reader's position* — but a single high-water id is fragile to out-of-order arrival and arm-time races. So the authoritative cursor is a reader-owned **processed-set** at `cursors/<role-slug>.done`: **unread = the order-independent set difference** (inbox ids minus `.done`), and the cursor advances **on action** (a message enters `.done` when acted on), not on receipt. The single-id `cursors/<role-slug>` is retained as a display/compat high-water; `.done` is seeded from it on first run. Both are single-author (reader-owned) — rule 2 stands untouched. No-message-lost follows directly: the drain is level-triggered off the durable set, so a missed or late wake re-surfaces the same unread on the next check.

### Activation — encoded interval, level-triggered
In-session activation is `heartbeat wait`: it polls unread (cursor-delta) on an **encoded low interval** — a single uniform rate (default ~5s), hard-max-clamped against a mis-set override. The cadence is out of the agent's hands (the silent-stall cause is encoded out, not left to judgment), and on `unread > 0` it exits, which the harness uses to **re-invoke the agent**. Detection is wired to activation, not to a log. The cursor advances on action only. *(An adaptive tight/rest split was originally specified but never built receiver-side — its only live effect was the idle rate, and polling is ~free, so a uniform low rate dominates it on cost and simplicity. The in-flight marker is retained — not for cadence, but for teardown-safety (`down` refuses an in-flight teardown) and the health verdict's observability.)*

### Teardown — announced dormancy ≠ a crash
`heartbeat down` retires a watch **by PID** (exact-arg identity; never an innocent recycled PID) and writes a **stand-down marker**. The marker is the difference between an announced dormancy and a crash, and it **overrides the lagging heartbeat age** in the verdict — a just-stopped watch reads DORMANT, not LIVE until its last beat ages out.

### Health verdict — marker-first
`heartbeat doctor` (and `status`) returns exactly one of: **LIVE** (fresh) / **DORMANT** (announced — a stand-down marker present) / **DEGRADED** (stale AND no marker = crash alarm). Marker-first precedence: the intentional announcement dominates the passive age. DEGRADED is the load-bearing verdict — it is how a crash gets *noticed*.

### Cross-death closure — what a pair can close itself, honestly bounded
In-session activation dies with the agent. Cross-death recovery is `heartbeat wake`: an **out-of-process, stateless one-shot** (run by a scheduled trigger) that, for each registered slug, detects a **stall** — NOT alive (heartbeat-age stale, or never started) AND no stand-down marker (a crash or an off pair, not an announced dormancy) AND `unread > 0` — and files a durable `wake-needed` marker plus an owner alert. The honest closure ladder:

- **No message is lost INDEPENDENTLY of the trigger** — that is the level-triggered durable cursor. `heartbeat wake` adds *timeliness*, never *safety*; a failed trigger costs latency, not a message.
- **who-supervises-the-supervisor TERMINATES.** The wake check is stateless, so there is no long-lived supervisor to keep alive — the regress ends at the OS/cloud scheduler that fires the trigger, not at another watcher.
- **Owner-relay wake** (durable marker + alert, via a local scheduled trigger) is available to any deployment today.
- **Genuine zero-human auto-wake** via a cloud-agent trigger is achievable but **topology-gated** — it requires the work surface be reachable from the cloud session; a local-only filesystem is not.
- **Full zero-touch wake of a LIVE interactive session is the harness boundary** (a wake-on-arrival hook). An external multiplexer key-injection (`tmux send-keys`) is a deployment-dependent option where the session is wrapped — not a clean primitive.

### Reliability as a falsifiable property — the failure-injection suite
A reliability claim is only as good as its falsification instrument. The reference implementation ships `heartbeat selftest`: a **deterministic failure-injection suite** that injects each failure mode (two-writer cursor swallow, empty/torn heartbeat, recycled PID, prefix-slug collision, the DEGRADED crash path, cross-death stall, boot self-verify race, …) on an **isolated** slug — live pairs untouched — and must return **identical-GREEN across N runs**. "Reliable" means this suite is reproducibly green, not that a happy path was observed once. A deployment claiming reliability provides and passes this suite (`conformance-criteria.md`).

### Activation backstop — the per-turn unread surface
The activation loop (`heartbeat wait`) only protects while it is *running* — and a signed mechanism that is not running protects nothing. The backstop is a **per-turn hook**: a `UserPromptSubmit` hook runs a fast, read-only check (reference helper `comms-unread-banner`) that injects the **cursor-delta** unread count — never inbox file-count (immutable mail never leaves the inbox, so file-count never drops) — into the agent's context at the **start of every turn**. So on *any* activation (owner prompt, `wait` re-invoke, session resume), the agent sees waiting mail without relying on memory or on the loop being up. It fires at the **turn boundary** — an agent mid-turn cannot ingest mail until the turn ends (the honest structural limit; the hook fires at the soonest point after). Hook and `wait` **compose**: `wait` re-invokes *between* turns, the hook surfaces unread *at* every turn. The reference helper is slug-agnostic by default (no args → all non-fixture slugs, each session reads its own line — zero-config) or slug-scoped per session where settings are directory-isolated; read-only, exit-0-advisory (never blocks a prompt), silent when clear.

*Superseded from v0.4.0:* the **commit-signal hooks** are retired — liveness-by-age plus the level-triggered durable cursor make a commit file-event unnecessary; a deployment MAY keep them but they are no longer a provision of this spec. The **scheduled wake companion** is generalized into `heartbeat wake`, now **liveness-aware** (it fires only on a true stall, not on every unprocessed message).

## The command surface (the contract)

A conforming deployment provides the following, parameterized by `<slug>` (from `comms.conf`):

| command | contract |
|---|---|
| `heartbeat up <slug>` | idempotent, singleton, fully-detached, self-verifying bring-up → GREEN/RED |
| `heartbeat wait <slug>` | in-session activation; exit on unread re-invokes the agent (encoded interval) |
| `heartbeat wake [slug…]` | out-of-process stall detection → durable `wake-needed` marker + owner alert |
| `heartbeat down <slug>` | safe teardown by PID; writes a stand-down marker |
| `heartbeat doctor <slug>` | health verdict LIVE / DORMANT / DEGRADED (marker-first) |
| `heartbeat status <slug>` | one-line liveness-by-age |
| `heartbeat gate <slug>` | boot-complete gate: assert live + armed-listening + drained → READY / NOT-READY (session-scoped) |
| `heartbeat selftest` | the deterministic failure-injection suite (the reliability proof) |

The reference implementation lives in the deployment's `.comms/bin/` and is **role- and project-agnostic** (self-locating: it resolves its own `.comms` from its install path, so the same `bin/` serves any project unchanged). Each project registers its slugs in `comms.conf`.

## Setup: an auditor duty

Standing the channel up belongs to the **auditor**, executed at its first boot (and re-verified at every boot):

1. If `.comms/` is absent: create the tree (README instantiated from this spec, both inbox directories).
2. File the **activation directive** to the builder's inbox — a `notice` that states: where the builder's inbox is, where the builder writes, the immutability/reply-link rules, the session-start read-then-bring-up discipline, and the corpus boundary. The builder learns the protocol from the channel's own first message.
3. The builder's **bootstrap pointer** — the one line that tells the builder to read its inbox at session start — is added to the builder's session-entry configuration (CLAUDE.md or equivalent) during the routing step of adoption (adoption guide, Step 8). Embedding the comms discipline in the audited role file's Operational Constraints as well is optional hardening, preserving the audited role file's single-required-edit property.
4. Where the reliability provisions are adopted: provide `.comms/bin/` (the heartbeat implementation), `comms.conf` (the slug registry), and the `cursors/` + `heartbeats/` directories; the channel is brought up per session via `heartbeat up <slug>` (rule 5). The cross-session wake **trigger** (the scheduled job that runs `heartbeat wake`) and its **project list** are **deployment-local configuration** — outside this spec, and deliberately *not* part of the kit: a deployment's project list may include private projects whose names must never appear in a shared spec.

## Housekeeping: an auditor duty, self-triggering

The auditor is the mail system's **sole housekeeper**. The duty rides the auditor's mandatory session-start read — it fires without prompting:

1. **Trigger:** an inbox directory exceeds **20** message files (adopters may tighten; do not loosen past the point where cold-start listing hurts).
2. **Action:** move **answered** messages, except the 10 newest files, to `.comms/archive/<inbox-name>/<year>/`. **Unanswered messages are pending work and are NEVER archived**, regardless of age.
3. **Relocation only.** Content is never edited during a sweep; immutability survives archiving.
4. **Sweep notice** filed to the builder's inbox listing the ids moved. Sweep notices are the housekeeping log; no separate index is maintained.
5. **Id resolution survives moves:** ids equal filename basenames; `re:` links and corpus references cite ids, not paths. To resolve an id, search `.comms/` including `archive/`.

## Multi-role projects

Inbox-per-recipient generalizes: each role gets an inbox; writers are everyone else plus the owner; filenames carry the sender. Each role registers its own slug in `comms.conf`; the same `bin/` serves all. Lanes may migrate incrementally — a project may run inbox lanes between builder and auditor while other roles remain on legacy conventions, provided the seam is documented in the project's comms README and legacy read paths are left undisturbed until their owners migrate.

## What this spec does not cover

- **Transport between machines** (the tree lives on whatever filesystem the roles share; remote sync is project infrastructure).
- **The cross-session wake trigger's scheduling and project list** — deployment-local (a deployment may watch private projects whose names never enter this shared spec).
- **Continuous work-surface monitoring by the auditor** — a posture concern (a `continuous-monitoring` auditor arms its *own* watch per its role file). Such a watch is **role-specific and stays OUT of the universal substrate**: e.g., an auditor's `*.role.md` artifact-watch is a separate single-responsibility process, so the substrate's single heartbeat writer stays honest (a role-specific duty must not double-write the heartbeat).
- **The wake-on-arrival hook** (machine configuration). The per-turn unread *surface* is now a provision (§Activation backstop, reference helper `comms-unread-banner`) — it fires at the next turn boundary. What remains outside this spec is the **wake-on-arrival hook**: the only thing that could wake a *live* interactive session **mid-task**, crossing the mid-turn wall the cross-death ladder leaves to the harness.

# Change Control

Update version and provenance on every change.

## Provenance
- source: Generalized from the Aire RoleSmith pair's operational comms protocol (itself adapted from the Sketch team's single-writer outbox model, whose single-author and append-only invariants are retained), after operational use in multiple projects.
- time: 2026-06-11
- summary: v0.6.0 (2026-06-23) — **§Reliability: the wake-marker (armed-listening) added** — a new liveness mechanism (minor bump per `auditor-pattern-spec.md` §Versioning; additive — v0.5.x channels remain conformant). Liveness-by-age proves the *watch* alive but not the agent-waking *wake* armed (GREEN ≠ listening; `up` ≠ armed): the armed wake now stamps a tight-freshness, **session-bound** wake-marker (`<run>/<slug>.wake`) every poll, so **armed-listening** is as falsifiable as liveness, a quietly-died wake becomes DETECTABLE (closing the bring-up directive's "an event-wake that quietly died is indistinguishable from no mail"), and a new **`heartbeat gate`** asserts live + armed + drained before "boot complete" (session-scoped; cross-leg readiness uses the freshness-floor). Folded reference-impl hardening: `cb_mark_processed` rejects an empty id (the silent drain no-op); the display cursor labeled lexical-max/display-only (authoritative unread = `cb_unread_count`). **Proven-then-written:** built + run in a deployment by a decorrelated builder↔auditor pair — `heartbeat selftest` extended to 43 checks incl. the production enacting path (omitted-arg session binding), the gate's armed-fail path, and the freshness bracket (identical-GREEN); the binding verified live and the wiring auditor-CONFORMANT before this entry. The message protocol + all prior provisions are unchanged; SessionStart auto-bring-up is a deployment convenience (firing verified per-deployment), floored by the per-turn surface. v0.5.2 (2026-06-15) — **§Activation backstop added: the per-turn unread hook.** The activation loop only protects while running — a signed mechanism not running protects nothing (a running auditor with `comms-wait` down missed mail ~20 min and misdiagnosed the direction). Added a per-turn `UserPromptSubmit` hook (reference helper `comms-unread-banner`) that injects the **cursor-delta** unread count (never file-count — immutable mail never leaves the inbox) into context every turn, so an agent notices waiting mail at the next turn boundary regardless of whether the loop is up. Fires at the turn boundary (mid-turn ingest is the honest structural limit); composes with `wait` (between-turn) as belt+suspenders. Slug-agnostic (zero-config) or slug-scoped (per-session, settings-isolated); read-only, exit-0-advisory, silent-when-clear. The per-turn unread *surface* moves from §"does not cover" to a provision; the **wake-on-arrival hook** (live mid-task inject) remains the harness boundary. No message-protocol or criteria changes. v0.5.1 (2026-06-15) — **§Activation: poll interval COLLAPSED to a single uniform low rate** (default ~5s, hard-max-clamped), replacing v0.5.0's "bounded-adaptive tight/rest" language. Per the running auditor's triangulated refutation: the adaptive split's receiver-side tight-entry was never built (the in-flight marker's sole runtime writer is the selftest), so the only live effect was the idle rate; polling is ~free (pure bash, no model tokens), so a uniform low rate dominates on cost + simplicity. **Surgical** — only the cadence branch was removed from the reference `cb_wait_interval`; the in-flight marker is RETAINED for its other two consumers (teardown-refusal guard + health-line observability), verified before the delete. Reference selftest gains an `interval-uniform-ignores-marker` regression guard. No message-protocol or criteria changes; v0.5.0 channels remain conformant. v0.5.0 (2026-06-15) — **Reliability evolution (additive; the message protocol is unchanged).** The *watch economy* (v0.4.0) is evolved into **§Reliability**: liveness is now **by heartbeat age** (a passive stamp, never process-presence — retiring the "who watches the watcher" regress), and the reader cursor gains an authoritative reader-owned **processed-set** (`cursors/<slug>.done`) so *unread* is an order-independent set difference that advances on **action, not receipt** (no message lost — level-triggered). Added: a unified self-locating command surface (`up` idempotent self-verifying singleton/detached bring-up; `wait` encoded bounded-adaptive in-session activation; `down` by-PID teardown with a stand-down marker; `doctor`/`status` marker-first verdict LIVE/DORMANT/DEGRADED; **`wake`** stateless out-of-process cross-death closure; **`selftest`** the deterministic failure-injection suite that must be identical-GREEN over N runs — reliability as a *falsifiable* property). Premises 4-5 added (liveness-by-age; no-message-lost). Rule 5 evolved (arm-a-watch → `heartbeat up`; cursor → high-water + `.done`). Layout gains `bin/`, `comms.conf`, `heartbeats/`, `cursors/<slug>.done`. Setup item 4 evolved (provide `bin/`+`comms.conf`; the wake **trigger** and its **project list** are deployment-local and explicitly NOT in the kit — private projects must never be named in a shared spec). §"does not cover" notes the auditor's continuous-monitoring watch as a role-specific process kept OUT of the universal substrate (no heartbeat double-write). **Superseded:** commit-signal hooks (liveness-by-age + durable cursor make them unnecessary); the scheduled wake companion (generalized into the liveness-aware `heartbeat wake`). The message protocol — message-per-file, immutability, reply-link state, thread closure, corpus boundary, finding schema, housekeeping — is RETAINED unchanged; this is an evolution of one layer plus an additive section, on a stable spine. Built builder→auditor (brainstorm→design→M1 CONFORMANT→M2 reliability-signed→cross-death closure), each gate independently reproduced by the auditor; deployed and live before specification (proven-then-written). v0.4.0 (2026-06-11) — Watch-economy provisions per the owner's token-efficiency directive, shaped by the RoleSmith Auditor's adversarial design review (`2026-06-11-design-review-findings-watch-economy`). Reader-owned cursors replace the proposed writer-maintained sentinel per her F2 inversion; watch-lifetime rule with thread-scoped watches bound to the scheduled wake companion as a package per her F3; commit-signal hooks with honest event scope and a session-start presence check per her F4; per-provision rationale per her F1; rule 7 (the id's date is a claim) per her F6. v0.3.0 — Initial specification: message-per-file inboxes with immutability and reply-link state; thread closure; session-start read-then-arm discipline; corpus boundary; auditor-owned setup with the activation-directive bootstrap; auditor-owned self-triggering housekeeping; multi-writer filename variant and incremental-migration seam for multi-role projects; pin-free body practice.
