---
name: triage-ticket
description: Scrutinize a user story / Jira ticket for quality and AI-readiness. Use when the user pastes a ticket, user story, or requirements text and asks to triage, review, scrutinize, grade, or check it before implementation. Input is free-form text (Jira-style), NOT JSON.
---

# Triage Ticket

Scrutinize a plain-text user story (e.g. a Jira ticket) and judge whether it is
clear, complete, and unambiguous enough to implement correctly — especially by an
AI agent that cannot infer missing details. Ambiguous stories cause hallucinated
business rules, generic design patterns, and wrong edge cases.

The input is **free-form text**, not JSON. Do not demand a JSON schema. Evaluate
the prose against the six elements below and rewrite it into a sharper ticket.
If the ticket may go to a coding agent rather than only a person, also score it
against the coding-agent readiness checklist in [[ticket-standards]] section 5 —
that bar catches failure modes INVEST and the six elements don't (missing
access/environment info, an untestable-by-machine Definition of Done, silent
duplicates, and business decisions left for the implementer to guess at) — and
check it against section 6 to see whether it should be a parent ticket with
sub issues instead of one flat ticket.

## Pipeline

- **Inputs:** a pasted/existing ticket, **or** the output of [[draft-ticket]]
  handed here for a quality gate.
- **Outputs:** a verdict (Ready / Needs work / Not implementable), a scorecard,
  and a sharpened rewrite.
- **Next step:** once Ready, offer `pm-agent` to post/update it. If it fails
  on scope (several stories in one), send it back through [[draft-ticket]] to
  re-slice.

## How to run

1. Read the pasted ticket. If nothing was pasted, ask the user for the ticket text.
2. Score each of the six elements (below) as ✅ present / ⚠️ weak / ❌ missing.
   If this ticket may be agent-bound, also score the coding-agent readiness
   checklist from [[ticket-standards]] section 5 the same way.
3. Flag every anti-pattern you find, quoting the offending phrase. This
   includes checking Linear (not just the pasted text) for an issue that
   already covers the same underlying scope under a different title or a
   different level of decomposition — a silent duplicate is as much a defect
   as a missing acceptance criterion.
4. Give a verdict: **Ready**, **Needs work**, or **Not implementable**.
5. Produce a rewritten ticket that fixes the gaps. Where a fact is genuinely
   unknown (a business rule only the author knows), list it as an open question
   rather than inventing it.

## The six elements every story should cover

1. **Intent** — the user-facing outcome with zero ambiguity. Must name explicit
   inputs and outputs, validation rules, and data-flow expectations.
2. **Acceptance criteria** — verifiable conditions, not prose. Each should be
   checkable (a tester or test could pass/fail it). Prefer Given/When/Then or a
   bulleted list of concrete pass conditions.
3. **Examples** — concrete input values and expected outputs / realistic cases.
   Examples disambiguate everything; most hallucinations disappear once examples
   are added. A story with no examples is suspect.
4. **Constraints** — what must NOT happen and the hard rules: allowed formats,
   uniqueness rules, size/length limits, character restrictions, permissions.
5. **NFRs** — non-functional requirements: latency, performance, security,
   privacy (e.g. "must not log PII"), payload limits, cost. Without these the
   implementation defaults to generic, possibly slow/insecure choices.
6. **Metadata** — id/title, author, and created/updated context so the story is
   traceable. (Often supplied by Jira itself — note if absent.)

## Quality rules

- **Intent must define inputs and outputs.** Reject "user can save profile
  easily" → require "user can update name, email, and avatar, with validation
  and persistence."
- **Acceptance criteria must be checkable.** Each line should be objectively
  pass/fail, not a feeling. "Works well" is not acceptance criteria.
- **Demand at least one concrete example** for any non-trivial behavior.
- **Constraints are non-negotiable.** If formats, limits, or uniqueness aren't
  stated, flag them as missing — do not assume defaults.
- **Say what should happen, not how.** Implementation details (specific tables,
  libraries, component names) in a story are a smell — flag them.
- **Edge cases must be named.** Empty input, max size, duplicate, unauthorized,
  concurrent action, failure/error path.

## Anti-patterns to flag (quote the phrase)

- ❌ **Vague prose / weasel words** — "easily", "simply", "just", "etc.",
  "and so on", "should work", "user-friendly". AI cannot infer the missing detail.
- ❌ **Missing examples** — no concrete grounding for the logic.
- ❌ **Mixing implementation into the story** — tells the AI *how* instead of
  *what should happen*.
- ❌ **Unstated constraints** — no formats, limits, uniqueness, or permissions.
- ❌ **Untestable acceptance criteria** — cannot be turned into a pass/fail test.
- ❌ **Multiple stories in one** — several unrelated outcomes bundled together;
  recommend splitting.
- ❌ **Silent duplicate or overlap** — another ticket already covers this
  scope, under a different title or a different decomposition.
- ❌ **Redefines an existing content type** — the ticket proposes a "new"
  Contentful (or similar) content type that already exists under a
  different name, without checking the actual current content model first.
- ❌ **Business decision left for the implementer** — a client-facing or
  scope tradeoff with no stated answer, that a coding agent would otherwise
  have to guess at rather than escalate.
- ❌ **No priority** — without it the ticket can't be sequenced by anyone
  besides the person who wrote it. This toolkit deliberately has no
  hour-estimation step, so don't flag a missing estimate as an issue.
- ❌ **Should be a parent with sub issues, not one flat ticket** — the ticket
  bundles more than one independently-verifiable build phase (a new content
  type, several components to build, distinct phases like data model →
  assembly → responsive states) that a coding agent can't finish and verify
  in one pass. This is different from "multiple stories in one" — the work
  belongs together under one parent, it just isn't one unit of work. See
  [[ticket-standards]] section 6.

## Output format

```
## Triage: <ticket title>

**Verdict:** Ready | Needs work | Not implementable

### Element scorecard
- Intent: ✅/⚠️/❌ — <note>
- Acceptance criteria: ✅/⚠️/❌ — <note>
- Examples: ✅/⚠️/❌ — <note>
- Constraints: ✅/⚠️/❌ — <note>
- NFRs: ✅/⚠️/❌ — <note>
- Metadata: ✅/⚠️/❌ — <note>

### Coding-agent readiness (if applicable)
- Access & environment named: ✅/⚠️/❌ — <note>
- Machine-checkable Definition of Done: ✅/⚠️/❌ — <note>
- Design-fidelity basis stated: ✅/⚠️/❌ — <note>
- Decisions reserved for a human: ✅/⚠️/❌ — <note>
- No duplicate/overlap on the board: ✅/⚠️/❌ — <note>
- No duplicate/overlapping content type: ✅/⚠️/❌ — <note>
- Milestone matches actual scope: ✅/⚠️/❌ — <note>
- Priority set: ✅/⚠️/❌ — <note>
- Right-sized, or already a parent with sub issues: ✅/⚠️/❌ — <note>

### Issues found
1. <issue> — quote: "<offending phrase>"

### Open questions for the author
- <fact that only the author can supply>

### Suggested rewrite
<the improved ticket text, prose with bulleted acceptance criteria and examples>
```

## Review checkpoint

The verdict and rewrite are a **draft for the human to review**.

1. Present the verdict, scorecard, and rewrite.
2. Explicitly ask the human to confirm or correct it — particularly any open
   questions you raised for the author, which only they can answer.
3. Fold in their input and show the revised version.
4. Only advance (post/update in Linear via `pm-agent`) once the human
   confirms.
