---
name: ticket-standards
description: The house standard for a well-formed agile work item — the Cohn user-story template, the INVEST quality check, vertical slicing, the Definition of Ready checklist, a coding-agent readiness bar for tickets an AI agent may implement unsupervised, and when a ticket should become a parent with sub issues instead of one flat ticket. Use when drafting, grooming, or reviewing a user story, bug, or task before it goes on the board, especially before handing it to a coding agent.
---

# Ticket Standards

The shared quality bar for work items on the board. Apply it when drafting a new
ticket, grooming the backlog, or reviewing a story before it is picked up. It
pairs with [[draft-ticket]] (turn a rough idea into a story) and
[[triage-ticket]] (grade an existing story).

## 1. The Cohn user-story template

Write user stories in the Mike Cohn form so intent, actor, and value are all
explicit:

> **As a** \<role/persona\> **I want** \<capability\> **so that** \<benefit\>.

- The **role** is a concrete persona, not "the user" — name who actually does this.
- The **capability** is a single user-facing outcome, not an implementation step.
- The **benefit** is the reason it is worth doing — if you can't state it, question
  whether the story belongs on the board.

Bugs and tasks don't need the Cohn form, but still need a clear title, the
observed-vs-expected behavior (bugs), and explicit acceptance criteria.

## 2. INVEST quality check

A good story is:

- **I**ndependent — can be built and shipped without being blocked by a sibling
  story; minimal ordering dependencies.
- **N**egotiable — describes the outcome, not a frozen implementation contract;
  leaves room for a conversation about *how*.
- **V**aluable — delivers observable value to a user or the business on its own.
- **E**stimable — clear enough that the team can size it; if it can't be
  estimated, it's under-specified or too big.
- **S**mall — fits comfortably inside a single iteration; if it doesn't, split it.
- **T**estable — has acceptance criteria a test (or a tester) can pass/fail
  objectively.

If a story fails an INVEST letter, name which one and fix it before it's Ready.

## 3. Vertical slicing

Slice stories **vertically** — each one delivers a thin, end-to-end piece of
user-visible value through every layer it touches (UI → API → data), not a
horizontal layer on its own.

- ✅ "As a member I can cancel a single upcoming booking" (touches the whole
  stack, shippable, demoable).
- ❌ "Build the bookings database table" / "Add the cancel button (no wiring)"
  — horizontal slices that deliver nothing usable alone.

If a story is really a horizontal slice, either fold it into the vertical story
it serves or re-cut the work so each ticket stands up on its own.

## 4. Definition of Ready checklist

A story is **Ready** to be pulled into a sprint only when all of these hold. If
any fail, the story stays in grooming — say which ones and what's missing rather
than filling the gap with an assumption.

- [ ] **Persona & value** — the Cohn "as a / so that" names a real actor and a
      real benefit.
- [ ] **Acceptance criteria** — present, and each one is objectively pass/fail
      (Given/When/Then or a concrete checklist), not "works well".
- [ ] **Example** — at least one concrete input → expected output, including a
      realistic edge case, for any non-trivial behavior.
- [ ] **Constraints** — hard rules stated: formats, limits, uniqueness,
      permissions, what must *not* happen.
- [ ] **Scope boundary** — what's explicitly out of scope is written down.
- [ ] **Dependencies** — known blocking dependencies are identified (or "none").
- [ ] **INVEST** — passes all six letters; if not, the failing letter is
      addressed.
- [ ] **Sized** — small enough to fit one iteration; larger stories are split
      first.

## 5. Coding-agent readiness

A story can pass INVEST and the human Definition of Ready above and still be
unsafe to hand to an autonomous coding agent. Apply this checklist whenever a
ticket may be picked up by an agent (Claude Code or similar) rather than only
a person — and check it during drafting or triage on any project where that's
a live possibility, not only when someone explicitly asks for an "agent
ready" grade.

- [ ] **Access and environment named** — which repo, which API or content
      model, and what credentials/environment variables the work needs are
      stated (or "uses existing project defaults"). An agent that has to
      guess where to look produces confident, wrong output instead of asking.
- [ ] **Definition of Done is machine-checkable** — at least one acceptance
      criterion the agent itself can verify without a human's eyes (a passing
      test, a successful build, a visual diff under a stated tolerance), not
      only "matches the design" or "looks right."
- [ ] **Design-fidelity basis is stated**, for any ticket whose acceptance
      criteria depend on a Figma (or similar) design: whether structured
      design data (Dev Mode, tokens, exact spacing) was available, or the
      ticket was built from screenshots/visual inspection only. A
      "pixel-perfect" bar with no structured data behind it is a bar the
      agent cannot actually verify itself, and should be called out as such
      rather than left implicit.
- [ ] **Decisions reserved for a human are called out by name** — anything
      that is a business, scope, or client-facing tradeoff (not a technical
      implementation choice) is listed under its own heading, and the agent
      is told to stop and ask rather than pick a default. A verbal client ask
      that never made it past a meeting is exactly the kind of thing that
      belongs here.
- [ ] **No duplicate or overlapping ticket already covers this scope** —
      checked against the board itself, not just against memory. A broad
      ticket and a decomposed set of tickets describing the same underlying
      build are as much a duplicate as two tickets with identical titles.
- [ ] **No duplicate or overlapping content type already exists**, for any
      ticket that defines, migrates into, or modifies a Contentful (or
      similar CMS) content type. Check the actual current content model
      first (a Contentful MCP/API if one is available, or the project's own
      content-model documentation and prior data-modeling tickets) before
      proposing a "new" type — a "Stats Card" and a "Stat/Metric Callout"
      defined separately by two different tickets is the same failure as a
      duplicate ticket, just one level down, and it's easy to miss because
      the two tickets don't look alike on the surface.
- [ ] **Milestone or grouping matches the ticket's actual scope** — the
      ticket's real content lines up with what the milestone or epic it's
      filed under says it's for. A "smoke test" milestone quietly holding
      real production build work is a readiness failure even when the
      ticket itself is well written.
- [ ] **Sequencing is clear from the ticket's place in the backlog**, not from
      a priority field. This project treats every ticket as equal priority,
      since all of it needs to be done — don't flag a missing priority as a
      readiness gap. What still needs to be unambiguous is which milestone or
      section the ticket belongs to, since that ordering is what actually
      tells a coding agent (or a person) what to pick up next.

If a ticket fails one of these, it can still be fine for a person to pick up,
but say explicitly that it is not ready to hand to an agent until fixed —
never let it through silently.

## 6. When a ticket should become a parent with sub issues

A ticket can pass every check above and still be too much for a coding agent
to pick up and finish in one verifiable pass. This is a different failure
from "multiple stories in one" (INVEST/Independent, and the triage
anti-pattern) — that's about unrelated outcomes bundled together and should
be split into separate, independent tickets. This is about **one page or
feature whose build genuinely has more than one independently-verifiable
phase**, where a parent ticket with sub issues (Linear's actual parent/child
relationship, not just a checklist in the description) is the right shape,
not a flat ticket and not several disconnected top-level tickets either.

Judge each ticket on what it actually contains — there is no fixed subtask
template or required count. Decompose when it hits one or more of:

- It introduces a **new or modified content type or field** in Contentful (or
  a similar CMS) — that's a distinct, independently-verifiable unit of work
  from assembling the page around it.
- It requires **more than one component to be built or substantially
  modified** — not just populated with content an existing component already
  accepts.
- It has **distinct build phases that are each independently verifiable** —
  e.g., content-model/data work, then assembly, then responsive or
  interaction states — where a coding agent could finish and verify one phase
  without the others being done.
- It's large enough that one coding-agent session realistically can't finish
  *and verify* it in a single pass.

Skip decomposition when none of these apply — say so explicitly rather than
leaving the human to wonder whether it was overlooked.

When decomposing, each sub issue must:

- Be **independently completable and verifiable** without its siblings
  finished first, unless a real build-order dependency exists — state that
  dependency explicitly (a blocking relation) rather than leaving it implied
  by list order.
- Meet **section 5 above on its own** — its own access/environment, its own
  machine-checkable Definition of Done, its own design-fidelity basis, its
  own decisions reserved for a human — not inherited by reference only.
- Have a **title specific enough to act on without re-reading the parent**,
  while its description links back to the parent issue for shared context
  (the design reference, decisions reserved for a human) instead of
  duplicating it.

## Rules

- Describe **what** should happen, not **how** to build it — keep tables,
  libraries, and component names out of the story body.
- Don't invent business rules to fill a gap. If only the author knows the
  answer, list it as an open question and ask rather than assuming a default.
- If a story bundles several unrelated outcomes, split it into separate stories
  before it's Ready.
- Never mark a story Ready with untestable acceptance criteria — that's the most
  common cause of wrong implementations.
