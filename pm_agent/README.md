# pm-agent

AVH Contentful migration toolkit for Claude Code — a Linear issue agent
centered on turning the Figma design file into a working backlog, plus a
small set of supporting ticketing skills, all held to a coding-agent
readiness bar so tickets are safe to hand to an autonomous coding agent, not
only a person.

> This is an AVH-specific fork of Mapleton Hill's general-purpose `pm-agent`.
> It's scoped down to Figma-driven ticket creation for this engagement; the
> feature-spec/prioritize-backlog/SOW/status-update/actuals-report skills from
> the general template, plus `estimate-ticket` (dropped from this fork — an
> hour estimate isn't something a coding agent consumes to do the work
> correctly, and it added a step that didn't pay for itself), were archived
> out of this copy rather than deleted — see
> [`_archived-pm-agent-skills/`](../_archived-pm-agent-skills) at the repo
> root if this engagement needs any of them back.

## What's inside

| Component | Type | What it does |
|---|---|---|
| `pm-agent` | agent | Orchestrates the skills below, end to end, and posts/updates issues in the AVH Contentful project in Linear. |
| `figma-to-tickets` | skill | The main entry point. Two modes: walk the AVH Figma file's pages/frames and draft one ticket per screen/component for a backlog, or point it at a single page and it drafts or updates the one main ticket for that page — grounded in what's actually in the design, never guessed. Decomposes a page into Linear sub issues when the build genuinely has more than one independently-verifiable phase. Also checks the existing Contentful content model before proposing a "new" content type. |
| `review-current-site` | skill | Audits the live aspenvalleyhealth.org site page by page — content, components, third-party integrations — so migration tickets account for what has to survive the move, not just what's in the new design. |
| `draft-ticket` | skill | Turns one slice or rough idea into a fully-specified story, interviewing you about the gaps first. |
| `triage-ticket` | skill | Grades an existing ticket for clarity and coding-agent readiness, and rewrites it. |
| `ticket-standards` | skill | The house quality bar — Cohn template, INVEST, vertical slicing, Definition of Ready, a coding-agent readiness checklist (access/environment named, a machine-checkable Definition of Done, no duplicate tickets or content types, milestone fit, priority set), and when a ticket should become a parent with sub issues instead of one flat ticket. |

## How the pieces fit together

Two sources of truth feed the backlog: the Figma designs (what the new site
should be) and the live site (what has to survive the move). `figma-to-tickets`
and `review-current-site` each ground tickets in one of those; `draft-ticket`
and `triage-ticket` handle everything that happens to an individual ticket
after that — refining one that needs more detail, or grading a pasted one.

```
  (Figma file/frame) ─► figma-to-tickets ─┐
                                          ├─► draft-ticket ─► triage-ticket
  (live site page)   ─► review-current-site ┘   more detail    grade & sharpen
                                                 on one ticket

  figma-to-tickets: batch mode drafts one ticket per screen/component,
  granularity confirmed with you first so shared components never get
  ticketed twice, checked against the actual Contentful content model so a
  component never gets a duplicate content type under a different name.
  Point it at a single page instead and it creates or updates just that
  page's ticket, decomposing it into Linear sub issues when the build has
  more than one independently-verifiable phase.
  review-current-site: flags what's on the live page today that the Figma
  frame is missing (or vice versa) before tickets are drafted.

  pm-agent posts & updates all of these in Linear (workspace colt-jones,
  team Colt Jones / COL, project AVH Contentful).
  triage-ticket grades any ticket pasted in from outside this flow.
```

Entry points are flexible: a Figma file/page link with a plural ask ("tickets,"
"the backlog") → `figma-to-tickets` batch mode; a single Figma page/frame link
with "draft/update the ticket for this page" → `figma-to-tickets` single page
mode; "what's actually on this page today" → `review-current-site`; a rough
single idea → `draft-ticket`; a pasted ticket → `triage-ticket`. Just describe
the goal and `pm-agent` routes to the right skill and offers the next one.
There is deliberately no estimation stage — see the note under Usage below for
why.

## Setup: connect Linear

This plugin no longer bundles its own MCP server — it uses Linear's native
connector instead, so there's no token to generate, scope, or store anywhere.

### 1. Connect Linear

In Claude Code or Cowork settings, find Connectors, then Linear, and connect
it. That's a normal sign-in-and-approve flow in your browser, nothing to copy
or paste.

### 2. Confirm the workspace, team, and project

This plugin assumes:

- **Workspace:** `colt-jones`
- **Team:** `Colt Jones` (key `COL`)
- **Project:** `AVH Contentful`

If any of these differ in your Linear account, update the names in
`pm_agent/agents/pm-agent.md` to match, or just tell pm-agent the right
team/project the first time you use it and it'll use that going forward for
the session.

That's the whole setup. No environment variables, no restart required beyond
what connecting the app itself needs.

## Live site access

`review-current-site` reads aspenvalleyhealth.org directly (via WebFetch or
whatever browsing tool is available in your session) every time it runs — it
never answers from a cached memory of the site, since pages change. No setup
is needed beyond normal internet access.

## Figma access

`figma-to-tickets` reads the AVH design file directly if a Figma MCP tool or
connector is available in your session. If not, it falls back to whatever you
paste or share in the moment (the page/frame outline, exports, screenshots) —
it will never invent frame content it hasn't actually seen. Either way, every
ticket it drafts states which kind of access it actually got (structured
design data vs. screenshots/manual inspection), since that changes how much
weight a "pixel-perfect" acceptance criterion can carry. The current AVH
file:

```
https://www.figma.com/design/haNEF3v9Zr1mF2fm6nWPya/AVH-IA---Content
```

## Run it with local code access for best results

For the sharpest output, **run Claude Code from inside the relevant repository
checkout** so the agent can read the actual codebase. The `pm-agent` agent and
the ticketing skills all have `Read`, `Grep`, and `Glob` access, and they use it
to ground their work in reality rather than guessing:

- **draft-ticket / triage-ticket** confirm which components, endpoints, and
  existing patterns a story touches — so acceptance criteria and constraints
  match how the system actually behaves instead of inventing plausible-sounding
  detail.
- **pm-agent** avoids fabricating context: it fills a story's technical detail
  from what it can actually read, not from assumption.

Without a local checkout the tools still work — they just fall back to what you
paste in, the Figma file, and Linear — but the results are weaker and more
assumption-driven. Point the session at the codebase the ticket concerns
whenever you can.

## Usage

The skills are **invoked by describing your goal in plain language**, not by
slash commands — `pm-agent` reads the request and routes to the right skill,
then offers the next stage in the pipeline. You don't name the skill; you say
what you want. Typical triggers:

| Say something like… | Runs | You get |
|---|---|---|
| *"turn this Figma page into tickets"* / *"build the backlog from the designs"* | `figma-to-tickets` (batch) | A batch of tickets, one per screen/component, grounded in the Figma file and checked against the current content model |
| *"draft the ticket for this page"* / *"update the ticket for this page"* + a Figma link | `figma-to-tickets` (single page) | One main ticket created or updated for that page, with Linear sub issues if the build has more than one independently-verifiable phase |
| *"what's actually on the current homepage?"* / *"audit the live site before we migrate this"* | `review-current-site` | A page-by-page inventory of the live site, plus gaps vs. the Figma design |
| *"draft a story for X"* / *"write a ticket for…"* | `draft-ticket` | One fully-specified, implementable story |
| *"triage this ticket"* (paste one) / *"is this ready?"* | `triage-ticket` | Verdict, element scorecard, and a sharpened rewrite |
| *"create this issue in Linear"* / *"update COL-42"* | `pm-agent` | The item posted/updated in Linear |

`ticket-standards` isn't invoked directly — the drafting and triage skills apply
it automatically as their quality bar.

**Why there's no estimation step:** an earlier version of this toolkit had an
`estimate-ticket` skill that put hours on a story. It's been dropped. An hour
figure isn't something a coding agent reads or acts on to do the work
correctly, and requiring one before a ticket counted as "ready" just added a
step and burned tokens without changing what the agent could actually
execute. Priority is different and is still required on every ticket — that's
the signal that decides what gets picked up next, whether by a person or by
whatever process is dispatching work to an agent. If you need hours for your
own budget tracking against the SOW, do that sizing separately, outside this
pipeline.

**How page tickets get decomposed:** `figma-to-tickets` and `draft-ticket`
judge each ticket on what it actually contains, not a fixed template. A page
that introduces a new Contentful content type, needs more than one component
built, or has distinct build phases a coding agent would finish and verify
separately (data model, then assembly, then responsive states, say) gets
broken into real Linear sub issues under one parent ticket — each held to the
same readiness bar as any other ticket, not just a checklist line. A page
that's just existing components being populated with content stays a single
ticket, and the skill says so explicitly rather than leaving you to guess
whether it considered splitting it. Pointing `figma-to-tickets` at a single
page you've already ticketed re-reads the design, diffs it against the
existing ticket and its sub issues, and proposes updates — it never silently
touches a sub issue that's already Done or In Progress.

### Chaining across the pipeline

You can also drive several stages in one go, staying in the loop between them:

> *"Turn the homepage and services frames into tickets, then triage each one
> against the current board."*

`pm-agent` runs `figma-to-tickets`, shows you the batch, and — once you approve
it — grades each ticket with `triage-ticket`. It won't auto-run the whole
chain unattended; the batch review and any bulk creation are decision points it
checks with you.

## Verify the output — you're the reviewer

Every skill produces a **draft for you to review**, not a finished artifact, and
there's a review checkpoint at the end of each stage. Nothing advances to the
next stage — and nothing is written to Linear — until you confirm. Treat
each hand-off as your cue to check the work and prompt for changes; silence is
not approval.

**You can, and should, verify everything, especially the Figma grounding.**
`figma-to-tickets` can misread a frame, miss a shared component, or miss that
a "new" content type already exists under a different name — always check
that a drafted ticket actually matches what's in the design (and the current
content model) before it's created. The skills are there to do the legwork
and show their reasoning; the sign-off is yours.
