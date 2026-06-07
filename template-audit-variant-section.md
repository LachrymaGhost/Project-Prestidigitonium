---
name: template-audit-variant-section
version: 0.1.0
status: draft
license: Apache-2.0
maintained_by: Aire System Architect (ASA)
domain_tags: [template, audit, governance]
references: auditor-pattern-spec.md v0.1.0
---

# Template: §Audit-Variant Section

This template provides a copy-pasteable skeleton for the §Audit-Variant section to be added to any role file conforming to `auditor-pattern-spec.md`. Placeholder content is marked with `<…>` tokens and explanatory notes in `> Note:` blocks. Replace every `<…>` token before considering the section complete.

Section placement in the host role file: under `# Change Control` → `## Provenance`, as a top-level `# §Audit-Variant — <Project> <Role> Auditor's lens on the <N> <rule-unit-name>`. The heading hierarchy keeps Provenance as a child of Change Control and §Audit-Variant as a peer of the role's other top-level sections (Operating Rules, Verification, etc.).

The template is reproduced below in literal copy-pasteable form. The same content with explanatory notes inline follows the literal block.

---

## Copy-pasteable skeleton

```markdown
# §Audit-Variant — <Project> <Role> Auditor's lens on the <N> <rule-unit-name>

**Architectural note:** This auditor instance follows the Project Prestidigitonium auditor pattern (`auditor-pattern-spec.md` v<X.Y.Z>). The auditor uses **this same role spec** with an audit lens — there is no separate `*-auditor.role.md` file. The role IS the capability; `audit` is one of two lenses (alongside `execute`) for applying it.

**Encompassment scope.** The auditor's knowledge is a proper superset of the builder's: the auditor reads everything the builder reads (the role file, the brief, the domain corpus, the governance specs) AND the asymmetric corpus described below. Authority chain bottoms out in upstream governance (`<list upstream governance specs>`) plus the asymmetric corpus, not in this role file's rules. The role file's rules are themselves auditable.

**Asymmetric corpus access.** The auditor loads the following materials at boot, which the builder MUST NOT load by default:
- `<path or convention for prior audit findings>`
- `<path or convention for failure post-mortems>`
- `<path or convention for drift catalogs>`
- The §Audit-Variant clauses below (the auditor's own self-referential corpus)
- `<any project-specific asymmetric materials>`

**Adversarial default.** The auditor's default verdict is **refuted**. Verdicts flip to **confirmed** only on evidence surviving an attempt to refute. Audit clauses below are phrased in the refutation frame: the auditor *attempts to refute* a claim, rather than verifying it.

**Independent re-derivation.** On key checks (enumerated below), the auditor MUST compute its own answer first, then compare. Reading the builder's output before deriving is not permitted on these checks. Key-check categories for this role: `<enumerate categories — e.g., re-measurement, re-classification, re-derivation of acceptance criteria>`.

**Authority precedence.** When the §Audit-Variant clause and the execute-mode rule body conflict, the execute-mode rule body is authoritative. When the role file's rule and upstream governance conflict, upstream governance is authoritative — the auditor flags the rule as the defect. When the auditor's verdict and the user's directive conflict, the user is authoritative.

**Scope limits acknowledged.** This pattern does not by itself resolve capability blind spots, adversarial-input failures, definitional disputes about what constitutes a defect, or sustained-load attention failures. See `auditor-pattern-spec.md` §"What this pattern does NOT solve."

## Audit Interpretation per <rule-unit-name>

### <Rule 1> — <Rule 1 short title> (audit interpretation)

The auditor attempts to refute the claim that <state the compliance condition for Rule 1, phrased as a refutable proposition>. Refutation mechanism: <state how the auditor independently establishes the answer — file checks, re-measurement, re-classification, governance reference lookup>. Audit fail: <state the conditions that constitute a confirmed defect — the refutation succeeded, or the claim cannot be confirmed against the mechanism>.

> Note: phrase every audit clause in the refutation frame. The grammatical choice is load-bearing per Mechanism 3. "The auditor verifies that X" biases toward confirmation; "the auditor attempts to refute X" biases toward catching defects.

> Note: state the refutation mechanism concretely enough that two independent auditors would converge. Vague mechanisms ("the auditor confirms the brief is correct") permit interpretation drift.

### <Rule 2> — <Rule 2 short title> (audit interpretation)

<repeat structure for each rule>

### <Rule N> — <Rule N short title> (audit interpretation)

<repeat structure for each rule>

## Cross-rule audit obligations

These obligations bind the auditor across the rule set, not against any specific rule. Minimum three required; five recommended.

1. **Independent verification posture** — The auditor MUST independently re-derive results in the role's enumerated key-check categories. <State the categories and the mechanism for each.> Trust-at-citation is insufficient.

2. **Scope ambiguity disclosure** — If the auditor pre-declares a HALT threshold (e.g., "any new artifact while BLOCKED"), the threshold MUST include explicit scope (which axis is BLOCKED, what conditions auto-clear, who can authorize override). Pre-declared HALT thresholds without scope language are governance defects.

3. **Authority chain check before HALT firing** — If an auditor's rule fires and the user is in active session, the auditor MUST poll the user's authorization status before posting the HALT. Posting a HALT against an action the user authorized off-channel is itself an audit defect.

4. **Stale rule-numbering refusal** — If a directive cites rule numbers that don't match the current authoritative role spec version, the auditor MUST flag the version mismatch and request re-issuance with current numbers.

5. **Memory-as-rule-source disclosure** — If the auditor applies a behavioral rule sourced from auto-memory or session-external state, and that rule materially affects an audit outcome, the auditor MUST cite the source by filename or identifier in the verdict.

<add project-specific cross-rule obligations as needed>
```

---

## Annotated version with explanatory notes

The same skeleton, annotated. Use this version to understand what each section is for; copy the unannotated version above into the host role file.

### Heading and architectural note

`# §Audit-Variant — <Project> <Role> Auditor's lens on the <N> <rule-unit-name>`

The H1 heading is the section anchor. `<Project>` and `<Role>` follow the naming convention in `auditor-pattern-spec.md`. `<N>` is the count of operating rules (or verification checks) the audit interpretation covers. `<rule-unit-name>` is the host role's terminology — "Operating Rules", "Verification Checks", "Normative Requirements" — whichever the host uses.

The architectural note states explicitly that this is a same-file lens, not a separate auditor role file. This forecloses a common misreading where adopters wonder if they need a `*-auditor.role.md` file. They do not.

### Encompassment scope preamble

This paragraph operationalizes Mechanism 1. It must:
- State explicitly that the auditor's knowledge is a proper superset of the builder's.
- Enumerate the upstream governance specs that form the auditor's authority bedrock.
- State that the role file's rules are themselves auditable against upstream governance.

The auditor's authority chain is the most-commonly-miswritten preamble element. Auditor authority does not bottom out in the role file's rules — that produces same-source blind spots. It bottoms out in upstream specs *plus* the asymmetric corpus.

### Asymmetric corpus access

This paragraph operationalizes Mechanism 2. It must:
- Name each asymmetric corpus source by path or convention.
- State that the builder MUST NOT load these materials by default.
- Include the §Audit-Variant clauses themselves as a self-referential corpus entry.

The "MUST NOT load" discipline is enforceable at adoption time (audit your role file's input list) and at runtime (when a builder loads the corpus inadvertently, flag the contamination).

### Adversarial default

This paragraph operationalizes Mechanism 3. It must:
- State the default verdict is **refuted**.
- State that verdicts flip to **confirmed** only on evidence surviving refutation attempts.
- Commit the audit clauses below to the refutation frame.

This is the most-commonly-overlooked mechanism in implementation. Drafting audit clauses in the verification frame is the natural English habit and produces auditors that confirm rather than challenge. The refutation frame must be intentional.

### Independent re-derivation

This paragraph operationalizes Mechanism 4. It must:
- Enumerate the key-check categories for this specific role.
- State that derive-then-compare is required for these checks.
- State that reading the builder's output before deriving is not permitted on these checks.

The key-check enumeration is role-specific and must be filled by the adopter. Generic key-check categories (re-measurement, re-classification, re-derivation of acceptance criteria) are starting points; role-specific additions belong here.

### Authority precedence

Standard three-level precedence:
1. Execute-mode rule body authoritative over audit-interpretation clause on lens conflicts.
2. Upstream governance authoritative over role file's rules.
3. User authoritative over auditor verdict.

These are not negotiable per `auditor-pattern-spec.md` §"Authority and precedence."

### Scope limits acknowledged

A one-line acknowledgment that the pattern does not cover capability blind spots, adversarial-input failures, definitional disputes, or sustained-load attention failures. Implementations that silently claim coverage beyond the pattern's scope are non-conformant.

### Audit Interpretation per <rule-unit-name>

One clause per rule. Each clause states:
- What the auditor attempts to refute (the refutable proposition).
- The refutation mechanism (concrete enough that two auditors would converge).
- Audit fail conditions (when the refutation succeeds, or when the claim cannot be confirmed).

Rules without an audit interpretation are unaudited. The §Audit-Variant section's coverage of the rule set should be **total** — every rule has a clause, even if the clause is "this rule is unauditable post-hoc; see Rule N for the executable check."

### Cross-rule audit obligations

A minimum of three obligations binding the auditor across the rule set. Five recommended. These cover behaviors that don't map to any single rule but are essential to audit integrity:

- Independent verification posture (mechanism enforcement).
- Scope ambiguity disclosure (pre-declared thresholds carry their scope).
- Authority chain check (don't HALT against user-authorized work).
- Stale rule-numbering refusal (audit against current versions only).
- Memory-as-rule-source disclosure (cite the asymmetric corpus when it drives verdicts).

Project-specific obligations belong here when a project has cross-cutting audit concerns not captured by per-rule clauses.

---

# Change Control

Update version and provenance on every change.

## Provenance
- source: Initial draft.
- time: 2026-06-06
- summary: v0.1.0 — Initial template for the §Audit-Variant section. Provides copy-pasteable skeleton with placeholder tokens plus an annotated companion explaining each preamble element, the per-rule clause structure, and the cross-rule obligation set. Aligned with auditor-pattern-spec.md v0.1.0; encodes all four mechanisms (encompassment scope, asymmetric corpus access, adversarial default, independent re-derivation) plus authority precedence and scope-limit acknowledgment as required preamble elements.
