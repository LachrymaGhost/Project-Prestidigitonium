---
name: comms-spec
version: 0.4.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [comms, mail, audit, governance, asymmetry]
references: auditor-pattern-spec.md, audit-corpus-spec.md, template-auditor-role-file.md
---

# Comms Specification — Builder ↔ Auditor Mail

## Purpose

This specification defines the **message-per-file inbox mail system** between an audited role (builder) and its auditor: how verdicts, audit requests, flags, and notices travel between the two without a human relaying them, how the channel stays cheap to cold-start, and how it maintains itself without prompting.

The design premises:

1. **A role cannot wake itself, but it can arm things that wake it.** Duties attach to the step that always runs (session start); the session-start step arms watches; the harness does all subsequent waking. The only recurring human act is one spin-up per session.
2. **Context burn is bounded by unanswered mail, not by history.** Answered messages are never re-loaded. There is no monolithic mail file to scan at cold start.
3. **The system bootstraps through its own first message.** The auditor stands the channel up and files the activation directive into the builder's inbox — the first message is the operating manual.

## Layout

```
.comms/
├── README.md            — the protocol, instantiated for the project (access rules + housekeeping)
├── inbox-<builder-slug>/  — read by the builder; written by the auditor and the owner
├── inbox-<auditor-slug>/  — read by the auditor; written by the builder and the owner
└── cursors/<role-slug>    — reader-owned position markers (§Watch economy)
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
5. **Session-start discipline (both parties):** read your inbox newest-first; act on messages without replies; **then arm a watch on your inbox directory per the watch-lifetime rule** (§Watch economy — session-long by default), so mid-session mail processes as events arrive. **After processing, update your reader cursor** (§Watch economy).
6. **Corpus boundary (load-bearing).** Mail is not corpus. The comms tree lives outside `audit-corpus/`, so the builder reading its own mail never touches auditor-only material (Criterion C-16 is undisturbed). Verdicts MAY cite corpus entries by id; the builder may then read only the cited entries, per the corpus access discipline.
7. **The id's date is a claim.** Verify it against the clock at filing — a date-stamped id filed without reading the clock is assertion-before-verification in its smallest unit (two independent same-day instances, one per party, are this rule's precedent). A misdated id is corrected by a follow-up message linking `re:`, never by rename.

## Watch economy — cost scales with events, not time

Each provision below buys a specific thing; the rationales are not interchangeable. The reader cursor persists the reader's position across sessions and keeps the new-mail check at shell level. The watch-lifetime rule removes the cost of a live session hosting watches through quiet periods. The scheduled wake companion removes the owner as scheduler — it is the only provision that wakes anyone without a session running. The commit-signal hooks improve wake latency for events file watches cannot see. None of them changes how verdicts read or how delivery is verified.

### Reader cursor
The newest message (the head) is derivable from the inbox directory itself — atomic with the filing act, zero added writes. What dies with sessions is the *reader's position*. Each reader maintains a cursor file at `.comms/cursors/<role-slug>` holding the id of the newest message it has processed, updated by the reader after processing and written by no one else — single-author by construction, so no co-edited file exists and rule 2 stands untouched. "Anything new?" is *newest inbox entry ≠ cursor*, checkable at shell level with no model context loaded. Writers maintain no freshness signal: a watcher trusting a writer-maintained signal is strictly weaker than a watcher reading the directory, and a signal update batched with the filing act is a claim unconditioned on the act's result — the delivery-claim defect class in infrastructure form.

### Watch lifetime
Default: watches are **session-long** (rule 5). A pair MAY adopt **thread-scoped** watches — arm while unanswered mail or an in-flight verification exists; disarm when the last thread closes — **only where the scheduled wake companion (or an equivalent cross-session wake mechanism) is deployed for that pair**. The two are a package: thread-scoped watches without a cross-session wake reinstate the owner as scheduler and leave quiet-period mail waiting unboundedly on the next session-start read. Scale by cadence — high-cadence pairs gain the most from thread scoping; pairs whose sessions sit far apart keep session-long watches and lean on the companion.

### Scheduled wake companion (deployment-owned; required for thread-scoped watches)
A scheduled job that compares each inbox head against its reader's cursor and exits silently on match; on mismatch it wakes the affected role or notifies the owner, per deployment. The quiet path reads cursors and directory listings only — no role context. Scheduling infrastructure is deployment-specific and outside this spec; its *existence* is inside this spec, as the precondition for thread-scoped watches.

### Commit-signal hooks (recommended)
Repo-local `post-commit` and `post-merge` hooks that touch `.comms/.commit-signal` give watchers a file event for commits — including commits of already-saved content, which modify no watched file. Honest event scope: `post-commit` fires on locally-authored commits, `post-merge` on pulls; a fetch-plus-reset fires neither. Hooks are unversioned configuration that fails silent, so installation is a setup-duty item and the auditor's session-start includes a hook **presence check** (one stat call) — absence becomes a detectable conformance fact instead of config faith. The hooks improve **wake latency only**: delivery claims still verify at the ref, always. Signal absence is evidence of nothing — the watch-blindness corollary is an evidence rule, not a tooling gap, and no hook closes it.

## Setup: an auditor duty

Standing the channel up belongs to the **auditor**, executed at its first boot (and re-verified at every boot):

1. If `.comms/` is absent: create the tree (README instantiated from this spec, both inbox directories).
2. File the **activation directive** to the builder's inbox — a `notice` that states: where the builder's inbox is, where the builder writes, the immutability/reply-link rules, the session-start read-then-arm discipline, and the corpus boundary. The builder learns the protocol from the channel's own first message.
3. The builder's **bootstrap pointer** — the one line that tells the builder to read its inbox at session start — is added to the builder's session-entry configuration (CLAUDE.md or equivalent) during the routing step of adoption (adoption guide, Step 8). Without it, the directive waits in an inbox no one knows to open. Embedding the comms discipline in the audited role file's Operational Constraints as well is optional hardening, at the adopter's discretion; it is not required, preserving the adoption's single-required-edit property for the audited role file.
4. Where the watch-economy provisions are adopted: create `.comms/cursors/`, install the commit-signal hooks (`post-commit`, `post-merge`), and run the hook **presence check** at every subsequent boot (§Watch economy).

## Housekeeping: an auditor duty, self-triggering

The auditor is the mail system's **sole housekeeper**. The duty rides the auditor's mandatory session-start read — it fires without prompting:

1. **Trigger:** an inbox directory exceeds **20** message files (adopters may tighten; do not loosen past the point where cold-start listing hurts).
2. **Action:** move **answered** messages, except the 10 newest files, to `.comms/archive/<inbox-name>/<year>/`. **Unanswered messages are pending work and are NEVER archived**, regardless of age.
3. **Relocation only.** Content is never edited during a sweep; immutability survives archiving.
4. **Sweep notice** filed to the builder's inbox listing the ids moved. Sweep notices are the housekeeping log; no separate index is maintained.
5. **Id resolution survives moves:** ids equal filename basenames; `re:` links and corpus references cite ids, not paths. To resolve an id, search `.comms/` including `archive/`.

## Multi-role projects

Inbox-per-recipient generalizes: each role gets an inbox; writers are everyone else plus the owner; filenames carry the sender. Lanes may migrate incrementally — a project may run inbox lanes between builder and auditor while other roles remain on legacy conventions, provided the seam is documented in the project's comms README and legacy read paths are left undisturbed until their owners migrate.

## What this spec does not cover

- Transport between machines (the tree lives on whatever filesystem the roles share; remote sync is project infrastructure).
- Continuous work-surface monitoring by the auditor (a posture concern — `continuous-monitoring` auditors arm additional watches per their own role files; this spec covers mail only).
- Harness-level hook backstops (machine configuration; recommended where available — a hook that surfaces the unread count each turn is structural enforcement of what rule 5 asks the model to remember).

# Change Control

Update version and provenance on every change.

## Provenance
- source: Generalized from the Aire RoleSmith pair's operational comms protocol (itself adapted from the Sketch team's single-writer outbox model, whose single-author and append-only invariants are retained), after one operational day in two projects.
- time: 2026-06-11
- summary: v0.4.0 (2026-06-11) — Watch-economy provisions per the owner's token-efficiency directive, shaped by the RoleSmith Auditor's adversarial design review (`2026-06-11-design-review-findings-watch-economy`), run over the mail system it revises. Reader-owned cursors replace the proposed writer-maintained sentinel per her F2 inversion: the head was always derivable from the directory; the cursor is the state that died with sessions; single-author by construction (no rule-2 exemption needed), and a writer signal batched with the filing act is a claim unconditioned on the act's result. Watch-lifetime rule added with thread-scoped watches bound to the scheduled wake companion as a package per her F3 (archive evidence: auto-disarm alone reinstates the owner as scheduler; unbounded quiet-period latency for low-cadence pairs). Commit-signal hooks added with honest event scope (post-commit misses pulls; post-merge named; fetch-plus-reset fires neither), setup-duty installation, session-start presence check, and the latency-only claim per her F4 (hooks are unversioned and fail silent; no hook closes the watch-blindness evidence rule). Per-provision rationale stated per her F1 (watch checks were already shell-level; each provision buys a different cost). Rule 7 (the id's date is a claim) per her F6: two independent misdated-id instances in one day, one per party. Rule 5 amended (watch-lifetime reference + cursor update); layout gains `cursors/`; setup duty gains item 4. v0.3.0 — Initial specification: message-per-file inboxes with immutability and reply-link state; thread closure; session-start read-then-arm discipline; corpus boundary; auditor-owned setup with the activation-directive bootstrap (builder's bootstrap pointer lands in session-entry configuration during the routing step, keeping the audited role file's single-required-edit property); auditor-owned self-triggering housekeeping (threshold sweep, relocation-only, sweep-notice log, move-safe id resolution); multi-writer filename variant and incremental-migration seam for multi-role projects; pin-free body practice.
