---
name: comms-spec
version: 0.3.0
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
└── inbox-<auditor-slug>/  — read by the auditor; written by the builder and the owner
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
5. **Session-start discipline (both parties):** read your inbox newest-first; act on messages without replies; **then arm a persistent watch on your inbox directory** for the remainder of the session, so mid-session mail processes as events arrive.
6. **Corpus boundary (load-bearing).** Mail is not corpus. The comms tree lives outside `audit-corpus/`, so the builder reading its own mail never touches auditor-only material (Criterion C-16 is undisturbed). Verdicts MAY cite corpus entries by id; the builder may then read only the cited entries, per the corpus access discipline.

## Setup: an auditor duty

Standing the channel up belongs to the **auditor**, executed at its first boot (and re-verified at every boot):

1. If `.comms/` is absent: create the tree (README instantiated from this spec, both inbox directories).
2. File the **activation directive** to the builder's inbox — a `notice` that states: where the builder's inbox is, where the builder writes, the immutability/reply-link rules, the session-start read-then-arm discipline, and the corpus boundary. The builder learns the protocol from the channel's own first message.
3. The builder's **bootstrap pointer** — the one line that tells the builder to read its inbox at session start — is added to the builder's session-entry configuration (CLAUDE.md or equivalent) during the routing step of adoption (adoption guide, Step 8). Without it, the directive waits in an inbox no one knows to open. Embedding the comms discipline in the audited role file's Operational Constraints as well is optional hardening, at the adopter's discretion; it is not required, preserving the adoption's single-required-edit property for the audited role file.

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
- summary: v0.3.0 — Initial specification: message-per-file inboxes with immutability and reply-link state; thread closure; session-start read-then-arm discipline; corpus boundary; auditor-owned setup with the activation-directive bootstrap (builder's bootstrap pointer lands in session-entry configuration during the routing step, keeping the audited role file's single-required-edit property); auditor-owned self-triggering housekeeping (threshold sweep, relocation-only, sweep-notice log, move-safe id resolution); multi-writer filename variant and incremental-migration seam for multi-role projects; pin-free body practice.
