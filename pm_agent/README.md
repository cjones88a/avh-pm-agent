# pm-agent

AVH Contentful migration toolkit for Claude Code — a Linear issue agent
centered on turning the Figma design file into a working backlog, plus a
small set of supporting ticketing skills, all held to a coding-agent
readiness bar so tickets are safe to hand to an autonomous coding agent, not
only a person. Built for developers who already have Figma MCP and
Contentful MCP connectors set up — see **Setup** below.

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
| `figma-to-tickets` | skill | The main entry point. Two modes: walk the AVH Figma file's pages/frames and draft one ticket per screen/component for a backlog, or point it at a single page (the "build a page" workflow, see below) and it drafts or updates the one main ticket for that page — grounded in what's actually in the design, never guessed. Decomposes a page into Linear sub issues when the build genuinely has more than one independently-verifiable phase, including a content-model subtask and a dependency check, and splits QA into content-parity vs. design pixel-fidelity subtasks. |
| `content-model-review` | skill | Checks a page's content blocks against the real, current Contentful content model (read through the Contentful MCP connector) and decides per block whether it's reused as is, extended, or genuinely new — then drafts specs for whatever's missing. Called by `figma-to-tickets` and `draft-ticket` before either one defines a content type; can also be run directly. |
| `review-current-site` | skill | Audits the live aspenvalleyhealth.org site page by page — content, components, third-party integrations — so migration tickets account for what has to survive the move, not just what's in the new design. |
| `draft-ticket` | skill | Turns one slice or rough idea into a fully-specified story, interviewing you about the gaps first. |
| `triage-ticket` | skill | Grades an existing ticket for clarity and coding-agent readiness, and rewrites it. |
| `ticket-standards` | skill | The house quality bar — Cohn template, INVEST, vertical slicing, Definition of Ready, a coding-agent readiness checklist (access/environment named, a machine-checkable Definition of Done, no duplicate tickets or content types, milestone fit against the real AVH01–AVH22 proposal codes), and when a ticket should become a parent with sub issues instead of one flat ticket. This project has no priority field — see **No priority field, by design** below. |

## How the pieces fit together

Two sources of truth feed the backlog: the Figma designs (what the new site
should be) and the live site (what has to survive the move). `figma-to-tickets`
and `review-current-site` each ground tickets in one of those; `draft-ticket`
and `triage-ticket` handle everything that happens to an individual ticket
after that — refining one that needs more detail, or grading a pasted one.

```
  (Figma file/frame) ─► figma-to-tickets ─┬─► content-model-review (Contentful MCP)
                                          ├─► draft-ticket ─► triage-ticket
  (live site page)   ─► review-current-site ┘   more detail    grade & sharpen
                                                 on one ticket

  figma-to-tickets: batch mode drafts one ticket per screen/component,
  granularity confirmed with you first so shared components never get
  ticketed twice. Point it at a single page instead (a live URL) and it
  runs the full "build a page" workflow below: find/create the ticket,
  audit the live site, read Figma for styling, run content-model-review,
  check dependencies, split QA into content-parity and design-fidelity
  subtasks, and log everything against the real AVH01–AVH22 milestone.
  content-model-review: checks page content against the real Contentful
  model (through the Contentful MCP connector) so a component never gets a
  duplicate content type under a different name.
  review-current-site: flags what's on the live page today that the Figma
  frame is missing (or vice versa) before tickets are drafted.

  pm-agent posts & updates all of these in Linear (workspace colt-jones,
  team Colt Jones / COL, project AVH Contentful).
  triage-ticket grades any ticket pasted in from outside this flow.
```

Entry points are flexible: a Figma file/page link with a plural ask ("tickets,"
"the backlog") → `figma-to-tickets` batch mode; a single page URL with "build
this page" / "draft or update the ticket for this page" → `figma-to-tickets`
single page mode (the build a page workflow, see below); "what's actually on
this page today" → `review-current-site`; "what does this page need from
Contentful" → `content-model-review`; a rough single idea → `draft-ticket`; a
pasted ticket → `triage-ticket`. Just describe the goal and `pm-agent` routes
to the right skill and offers the next one. There is deliberately no
estimation stage and no priority field — see **No priority field, by design**
below.

## The build a page workflow

This is the canonical flow for turning one page into a properly-tracked
piece of work, and it's what `figma-to-tickets` runs in single page mode:

1. You give it the URL of the page to build.
2. It finds the existing Linear ticket for that page, or confirms there
   isn't one yet.
3. It runs `review-current-site` against the URL to see what has to
   survive the migration, and writes that into the ticket's specs.
4. It reads the matching Figma frame for styling and component specs,
   writing a subtask for any new/modified component.
5. It runs `content-model-review` to decide what the page needs from
   Contentful — reuse, extend, or genuinely new — and writes a subtask for
   anything that isn't a pure reuse.
6. It checks dependencies: reused content entries from other pages, links
   to pages that may not exist yet, integrations, missing assets.
7. It writes QA as two separate subtasks when both apply: content-parity
   QA under the page's own milestone, and design pixel-fidelity QA under
   milestone AVH22.
8. It never claims something was checked, built, or confirmed unless it
   actually was.
9. Anything still unresolved goes under Open Questions, not a guess.
10. Everything gets filed in Linear against a real AVH01–AVH22 milestone,
    so progress rolls up against the signed proposal, not just a page name.

Every stage above still has its own review checkpoint — nothing gets
written to Linear until you confirm the ticket (and any subtasks) it
produces.

## No priority field, by design

This project has no priority field, and none of the skills should add one.
Every ticket is treated as equal priority; what decides sequencing is where
a ticket sits in the backlog and which milestone (AVH01 through AVH22) it's
filed under — a decision made by the humans running the project, not a
priority value on the ticket. If you see a skill asking for or flagging a
missing priority, that's stale — the current versions leave it unset and
don't treat its absence as a defect. There's also deliberately no
hour-estimation step (see the note under Usage below); track hours against
the SOW separately, outside this pipeline.

## Milestones track against the real AVH01–AVH22 proposal codes

Every ticket's milestone should be one of the real phase codes from the
signed proposal (AVH01 through AVH22), not a placeholder or an
independently-invented number. If your Linear project also keeps
page-section milestones (Home, Services, Providers, etc.) alongside the
numbered ones, both can coexist — the numbered milestones are what let you
report progress against the proposal's billing phases, and the page-section
ones are what let you browse by page. Check `ticket-standards` section 5's
milestone check before filing anything, and don't invent a new numbered
milestone without confirming it against the real proposal codes first.

## Setup: connect Linear, Figma, and Contentful

This plugin no longer bundles its own MCP server for Linear — it uses
Linear's native connector instead, so there's no token to generate, scope,
or store anywhere. It's built assuming your Figma and Contentful connectors
are also set up, since `figma-to-tickets` and `content-model-review` read
those directly rather than falling back to pasted content.

### 1. Connect Linear, Figma, and Contentful

In Claude Code or Cowork settings, find Connectors and connect each of
Linear, Figma, and Contentful. Each is a normal sign-in-and-approve flow in
your browser — nothing to copy or paste.

### 2. Confirm the workspace, team, and project

This plugin assumes:

- **Workspace:** `colt-jones`
- **Team:** `Colt Jones` (key `COL`)
- **Project:** `AVH Contentful`

If any of these differ in your Linear account, update the names in
`agents/pm-agent.md` to match, or just tell pm-agent the right team/project
the first time you use it and it'll use that going forward for the session.

### 3. Confirm the MCP tool names

`agents/pm-agent.md`'s tool allowlist currently lists generic, guessed
prefixes for the Figma and Contentful connectors: `mcp__figma__*`,
`mcp__Figma__*`, `mcp__contentful__*`, `mcp__Contentful__*`. These were
written before real connectors were installed and confirmed against this
session, so check them against whatever your actual Figma and Contentful
MCP connectors are named once installed (a connected session's tool list,
or `ListConnectors`/`ListMcpResourcesTool`-style introspection, will show
the real prefix) and correct the `tools:` line in `agents/pm-agent.md` if it
doesn't match. Until corrected, the agent may not actually have access to
tools under a different real prefix even though the connector itself is
connected.

That's the whole setup. No environment variables, no restart required beyond
what connecting each app itself needs.

## Live site access

`review-current-site` reads aspenvalleyhealth.org directly (via WebFetch or
whatever browsing tool is available in your session) every time it runs — it
never answers from a cached memory of the site, since pages change. No setup
is needed beyond normal internet access.

## Figma and Contentful access

`figma-to-tickets` reads the AVH design file directly through the Figma MCP
connector (see Setup above) — this is the primary path now that developers
have it set up. Only if that connector is genuinely unavailable does it fall
back to whatever you paste or share in the moment (the page/frame outline,
exports, screenshots) — it will never invent frame content it hasn't
actually seen. Either way, every ticket it drafts states which kind of
access it actually got (structured design data vs. screenshots/manual
inspection), since that changes how much weight a "pixel-perfect" acceptance
criterion can carry. The current AVH file:

```
https://www.figma.com/design/haNEF3v9Zr1mF2fm6nWPya/AVH-IA---Content
```

`content-model-review` reads the actual Contentful space directly through
the Contentful MCP connector — this is its primary source (see Setup
above). It falls back to the maintained content-model reference doc, and
then to prior Linear tickets, only when the connector isn't available, and
says explicitly which source it used.

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
| *"turn this Figma page into tickets"* / *"build the backlog from the designs"* | `figma-to-tickets` (batch) | A batch of tickets, one per screen/component, grounded in the Figma file |
| *"build this page"* / *"draft or update the ticket for this page"* + a page URL | `figma-to-tickets` (single page) | The build a page workflow above, end to end: one main ticket, a content-model subtask if needed, a dependency check, and split QA subtasks, all filed against a real AVH milestone |
| *"what's actually on the current homepage?"* / *"audit the live site before we migrate this"* | `review-current-site` | A page-by-page inventory of the live site, plus gaps vs. the Figma design |
| *"what does this page need from Contentful"* | `content-model-review` | A per-block classification (reuse / extend / new) against the real content model, and specs for anything missing |
| *"draft a story for X"* / *"write a ticket for…"* | `draft-ticket` | One fully-specified, implementable story |
| *"triage this ticket"* (paste one) / *"is this ready?"* | `triage-ticket` | Verdict, element scorecard, and a sharpened rewrite |
| *"create this issue in Linear"* / *"update COL-42"* | `pm-agent` | The item posted/updated in Linear |

`ticket-standards` isn't invoked directly — the drafting and triage skills apply
it automatically as their quality bar.

**Why there's no estimation step or priority field:** an earlier version of
this toolkit had an `estimate-ticket` skill that put hours on a story, and a
required priority field on every ticket. Both have been dropped for this
project. An hour figure isn't something a coding agent reads or acts on to
do the work correctly, and a priority value doesn't add anything that
backlog position and milestone (AVH01–AVH22) don't already tell you.
Requiring either before a ticket counted as "ready" just added a step and
burned tokens without changing what the agent could actually execute. If you
need hours for your own budget tracking against the SOW, do that sizing
separately, outside this pipeline.

**How page tickets get decomposed:** `figma-to-tickets` and `draft-ticket`
judge each ticket on what it actually contains, not a fixed template. A page
that introduces a new Contentful content type, needs more than one component
built, has distinct build phases a coding agent would finish and verify
separately (data model, then assembly, then responsive states, say), or has
real dependencies (reused content, links to other pages, integrations) gets
broken into real Linear sub issues under one parent ticket — each held to the
same readiness bar as any other ticket, not just a checklist line. QA is
always split into a content-parity subtask (files under the page's own
milestone) and a design pixel-fidelity subtask (files under AVH22) when both
apply. A page that's just existing components being populated with content
stays a single ticket otherwise, and the skill says so explicitly rather than
leaving you to guess whether it considered splitting it. Pointing
`figma-to-tickets` at a single page you've already ticketed re-reads the
design, diffs it against the existing ticket and its sub issues, and
proposes updates — it never silently touches a sub issue that's already Done
or In Progress.

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
