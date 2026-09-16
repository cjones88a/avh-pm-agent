---
name: figma-to-tickets
description: Turn a Figma file, page, or single frame into fully-specified Linear issues — draft a whole backlog, or draft/update the one issue for a specific page — decomposing a page into sub issues when it's complex enough that a coding agent needs to pick off pieces one at a time. Use when the user shares a Figma link, asks to turn designs into tickets or a backlog, or asks to draft or update the ticket for one page.
---

# figma-to-tickets

The AVH Contentful migration is design-driven: the Figma file is the source of
truth for what gets built, and this skill is how it becomes a working backlog.
It runs in two modes — **batch** (a whole file or page becomes a set of
tickets) and **single page** (one specific frame becomes exactly one main
issue, created or updated in place, decomposed into sub issues if the work
warrants it). Either way, every ticket is grounded in what is actually in the
file, never in a guess about what a screen probably contains.

## When to use this

- **Batch:** the user shares a Figma file/page link and asks to "turn the
  designs into tickets," "build the backlog from Figma," or similar.
- **Single page:** the user shares one specific page URL (the live site page,
  or a Figma frame) and says "build this page," "draft/update the ticket for
  this page," or similar — singular, naming one page. This creates or
  updates exactly one main issue for that page. See **Single page mode: the
  build a page workflow** below for the exact step order this follows.
- A previously-created ticket set needs a delta pass after the designs changed
  (see **Step 7a: re-syncing after a design change** below) — this applies to
  both modes.

## Single page mode: the build a page workflow

When the human hands you one page (typically a live site URL) and asks to
build or update its ticket, run these in order. Each numbered stage below
maps to one of the detailed steps further down — this section is the
checklist; the steps below it are how to actually do each one.

1. **Get the page.** The human gives you the URL of the page to build.
2. **Find or create the ticket.** Check Linear for an existing issue covering
   this page before drafting anything (Step 4). If one exists, you're
   updating it (Step 4a), not starting fresh.
3. **Audit the live site.** Run [[review-current-site]] against the URL to
   see what content and behavior actually exists today and has to survive
   the move. Write this into the ticket's migration-scope specs.
4. **Check Figma for styling.** Read the corresponding Figma frame (Step 1)
   for the visual/component spec. If the page needs new or modified
   components, write that as its own subtask (Step 6).
5. **Check the content model.** Run [[content-model-review]] against the
   page's content blocks to decide what's reused as-is, extended, or
   genuinely new in Contentful. Write a subtask for any new/changed type
   (Step 6).
6. **Check dependencies.** Look for anything this page needs beyond its own
   content: reused content entries from another page (not just links —
   actual shared data, like a location or contact card defined elsewhere),
   links to pages that may not exist yet, third-party integrations, and
   assets that haven't been supplied. Note each as a dependency on the
   ticket, not silently assumed to be handled elsewhere.
7. **Write the QA subtask(s).** Split by kind, don't lump them into one
   generic "QA" line: a **content-parity QA** subtask (does the migrated
   content match the source, are the dependencies from step 6 actually
   wired up) stays under the same milestone as the page itself; a **design
   pixel-fidelity QA** subtask (does the build match Figma to the stated
   tolerance) belongs under milestone **AVH22** instead, since that's the
   proposal's dedicated design-audit phase, not the page's own migration
   phase.
8. **Never lie.** Every claim in the ticket traces back to something you
   actually read this pass — the live site, the Figma frame, the current
   Contentful model, or the human. If you didn't check something, say you
   didn't, don't imply you did.
9. **List the unknowns.** Anything genuinely unresolved after steps 1 to 8 —
   an ambiguous Figma comment, a business decision only the client can make,
   a dependency you couldn't verify — goes under **Open questions**, not
   filled in with a guess.
10. **Log it in Linear, organized against the AVH codes.** The ticket (and
    each subtask) gets filed to project `AVH Contentful`, and its milestone
    is one of the real proposal codes AVH01 through AVH22 — check
    [[ticket-standards]] section 5's milestone check and the project's
    milestone reference before filing, never a placeholder or a guess at
    which code fits. Page-section milestones (Home, Services, etc., if still
    in use) track alongside this, not instead of it — check with the human
    which scheme is current before filing if it's unclear.

## Step 1: Read the file

A Figma MCP connector is expected to be set up in this session (see the
README) — use it as the primary path: enumerate the file's pages and frames,
and read each frame's content, component instances, and comments directly.
Only if that connector is genuinely unavailable, fall back to asking the user
to paste the page/frame outline (Figma's left-hand layers/pages panel), share
exports or screenshots of the frames in scope, or grant access another way.

**Note which kind of access you actually got**, and carry that forward into
every ticket drafted from it: structured design data (Dev Mode MCP, exact
tokens/spacing/variants) versus screenshots, a shared browser session, or
manual reading of the layers/properties panels. This isn't just a caveat for
this step — it becomes each ticket's design-fidelity basis (see Step 5 and
[[ticket-standards]] section 5), because a "pixel-perfect" acceptance
criterion means something different, and is verifiable by a coding agent to a
different degree, depending on which kind of access produced it.

Either way: **never invent a frame's name, content, or behavior you have not
actually seen.** If something is ambiguous or not visible in what you were
given, say so and ask rather than filling the gap with a plausible-sounding
guess. This matters more here than almost anywhere else in the pipeline —
every ticket's acceptance criteria trace back to what's actually on the canvas.

## Step 2: Determine the mode

- If the link is to a whole file or a page containing multiple frames, and the
  ask is plural ("tickets," "the backlog," "a sprint's worth") → **batch
  mode** → continue to Step 3.
- If the link is to one specific frame/page and the ask names it singularly
  ("the ticket for this page") → **single page mode** → skip Step 3 (there's
  nothing to batch — the unit of work is fixed) and go straight to Step 4.

## Step 3: Decide granularity with the human (batch mode only)

Before drafting anything, confirm the unit of work with the user via
AskUserQuestion (or, if unavailable, ask directly in plain language). The
options are typically:

- **Per page/screen** — one ticket per distinct page template (e.g. "Homepage,"
  "Service Detail").
- **Per reusable component/pattern** — one ticket per component that appears
  across multiple screens (e.g. "Hero," "Card Grid," "CTA Banner"), built once
  and reused.
- **Both, but not double-counted** — screens get tickets for page-level
  assembly and content, while shared components get their own separate tickets;
  a screen's ticket should reference the component tickets it depends on rather
  than re-describing that component's build work.

Default to "both, but not double counted" when the user has no strong
preference, since it mirrors how a component-based CMS build (Contentful, in
this case) actually gets built. Whatever is chosen, hold to it for the whole
batch — don't let some screens get component-level tickets and others not
without saying so.

**Watch for the double-counting failure mode:** if a component (say, a "Team
Member Card") appears on both the "Who We Are" and "Leadership" screens, it
gets exactly one component ticket, not one per screen it appears on. Before
drafting, group frames by the components they share and flag any overlap to
the user.

## Step 4: Check for an existing issue and content types

Before drafting anything new, check Linear for an issue that already covers
a screen or component in scope, so the same design element never gets a
duplicate ticket. This includes a ticket that describes the same underlying
build under a different name, a different granularity, or filed against a
milestone whose own stated purpose doesn't match what's actually being asked
for (e.g. real production build work landing in what's meant to be a
throwaway validation milestone) — not just an identical title.

- **Batch mode:** surface any matches to the user and ask whether to update
  the existing ticket instead of creating a new one.
- **Single page mode:** this check IS the fork. If no issue exists for this
  page, continue to Step 5 as a create. If one already exists, go to **Step
  4a** instead — you are updating it, not drafting from scratch.

Separately, for any screen/component whose ticket will define or migrate
into a Contentful content type, run [[content-model-review]] before
drafting — don't re-derive the content-model check inline here. That skill
reads the real, current model through the Contentful MCP connector and
decides per block whether it's reused as-is, extended, or genuinely new. A
component that looks new in Figma (say, a "Stats Card") can already exist in
Contentful under a different name (say, "Stat/Metric Callout") — ticket the
reuse, not a redefinition, and flag the naming mismatch to the user rather
than quietly picking one name.

### Step 4a: Update path (single page mode, issue already exists)

Do not re-draft the ticket from scratch — diff it against the current design.

1. Re-read the frame's current state (Step 1).
2. Fetch the existing issue (`get_issue`) and its sub issues, if any
   (`list_issues` with `parentId` set to this issue).
3. Compare the frame against what the issue currently says. Propose a
   **patch**, not a wholesale rewrite — use `save_issue`'s `patch`
   operations so unrelated history in the description isn't destroyed. Call
   out specifically what changed in the design and what that changes in the
   ticket (new acceptance criteria, an updated design-fidelity basis, a
   decision that's now resolved, etc.).
4. Compare against the existing sub issues (if this issue has any): propose
   **new** sub issues for work the design now requires that no existing sub
   issue covers, and **flag** (never auto-cancel) any sub issue whose
   corresponding design element appears to have been removed. Never touch a
   sub issue that's already Done or In Progress — surface a note about it
   instead of editing or reopening it, since the person working it may know
   something the diff doesn't.
5. Present the diff — not a full re-draft — for review before writing
   anything (Step 7).

## Step 5: Draft the ticket

For each screen or component confirmed in Step 2/3 (or the single page in
single-page mode), draft a ticket to the same bar as **[[draft-ticket]]** and
**[[ticket-standards]]** — Cohn template, INVEST check, vertical slicing,
Definition of Ready. Ground every element of the ticket in the Figma file:

- **Title** — the screen/component name as it appears in Figma (or a clear,
  consistent rename if the Figma name is unhelpful — note the rename).
- **Intent** — what this screen/component does for the user, inferred only from
  what's visible (layout, copy, evident interaction states) — not assumed
  business logic.
- **Acceptance criteria** — content blocks, states (default/hover/empty/error
  if shown), responsive behavior if multiple frame sizes are given, and any
  Contentful content-model implications (what becomes an editable field vs.
  fixed markup).
- **Design reference** — the direct Figma link (file + node id) for this
  screen/component, so the ticket always points back to its source of truth.
- **Design-fidelity basis** — whether this ticket's acceptance criteria came
  from structured design data or from screenshots/visual inspection only (see
  Step 1). State it plainly rather than letting a "pixel-perfect" criterion
  imply a precision the source material can't actually support.
- **Decisions reserved for a human** — any business, scope, or client-facing
  choice visible in the design that isn't purely a visual/technical fact (for
  example, whether a component's structure should stay editor-configurable
  later, if that came up outside the file itself). List it rather than
  quietly picking an answer.
- **Open questions** — anything ambiguous in the design, plus any actual design
  comments found in Figma (quote them, don't paraphrase into something more
  definite than they are).

## Step 6: Decide whether this ticket needs sub issues

A page ticket can be well-formed and still be too much for a coding agent to
pick up and finish in one verifiable pass. Judge **each page on what it
actually contains** — there is no fixed subtask template or required count.

Decompose into sub issues when the page hits one or more of:

- It introduces a **new or modified Contentful content type or field** —
  that's a distinct, independently-verifiable unit of work from assembling
  the page around it (this is the [[content-model-review]] output, see
  Step 4 — write it up as its own subtask rather than folding it into the
  page ticket's body).
- It contains **more than one component that must be built or substantially
  modified** (not just populated with content it can already accept).
- It has **distinct build phases that are each independently verifiable** —
  e.g., content-model/data work, then page assembly, then responsive or
  interaction states — where a coding agent could finish and verify one phase
  without having done the others.
- It's large enough that one coding-agent session realistically can't finish
  *and verify* it in a single pass.
- It has **real dependencies** beyond its own content — a reused content
  entry from another page, a link to a page that doesn't exist yet, a
  third-party integration, a missing asset — worth tracking as its own
  dependency note or subtask rather than left implicit in the page ticket.

**Every page ticket also gets a QA pass, split by kind, not lumped
together:** a **content-parity QA** subtask (migrated content matches the
source, dependencies are actually wired up) files under the page's own
milestone; a **design pixel-fidelity QA** subtask (matches Figma to a stated
tolerance) files under milestone **AVH22** instead — these are different
checks against different bars and belong to different phases of the
proposal, even when they're both "QA" on the same page.

**Skip decomposition of the non-QA work** when the page is just existing,
already-built components being populated with content and none of the above
applies — keep the build itself as one ticket (the QA split above still
applies). Say explicitly that you considered sub issues and why you didn't
propose any beyond QA, so the human isn't left wondering whether you forgot.

When you do decompose, propose sub issues sized to what *this* page actually
needs. Each one must:

- Be **independently completable and verifiable** by a coding agent without
  needing its siblings finished first — unless a real build-order dependency
  exists, in which case state it explicitly (a blocking relation between the
  issues), don't leave it implicit.
- Meet **[[ticket-standards]] section 5 on its own** — access/environment,
  a machine-checkable Definition of Done, design-fidelity basis, decisions
  reserved for a human — not inherited by reference from the parent alone.
- Have a **title specific enough that a coding agent knows what to do without
  re-reading the whole parent ticket**, while its description links back to
  the parent issue for the shared design reference and any decisions reserved
  for a human, rather than re-pasting the parent's full content.

## Step 7: Review checkpoint

Present the ticket(s) together — not one at a time in batch mode — so the
user can review consistency across the set (naming, granularity, any missed
overlap), along with any proposed sub issues and, in the update path, the
diff from Step 4a. Explicitly ask whether it needs changes; silence is not
approval.

Only after the user confirms does `pm-agent` write to Linear, using the
Linear MCP tools (`save_issue`) — never fabricate an issue ID or claim an item
was created or updated if the tool call didn't succeed. Create the main issue
first, capture its id, then create each sub issue with `parentId` set to that
id. In the update path, apply the confirmed patch to the existing issue and
create only the newly-approved sub issues; never delete or silently alter an
existing sub issue's state.

### Step 7a: re-syncing after a design change

When the user comes back after the Figma file has changed, don't re-draft the
whole batch. Re-read the affected pages/frames, diff against what the existing
tickets (and their sub issues) describe, and propose only the deltas: new
tickets or sub issues for new screens/components/work, updates for changed
ones, and a flagged (never auto-closed) list for anything that appears to have
been removed from the design. This is the same mechanism as Step 4a, run
across a batch instead of one page.

## Output format

Present each main ticket as:

```
### <Title>
**Type:** Component | Pattern | Task
**Milestone:** <AVH01–AVH22 code, per the milestone reference>
**Design reference:** <Figma link with node id>
**Design-fidelity basis:** <structured design data | screenshots/visual inspection only>
**Intent:** <1–2 sentences>
**Acceptance criteria:**
- ...
**Decisions reserved for a human:**
- ... (or "None identified")
**Open questions:**
- ...
**Sub issues:** <"None — page only assembles existing components with content" | a list below>
```

If sub issues are proposed, list each as:

```
  - **<Sub issue title>**
    **Depends on:** <sibling sub issue, or "None">
    **Acceptance criteria:** ...
    **Design-fidelity basis:** ...
    **Decisions reserved for a human:** ... (or "None identified")
```

For an update (Step 4a), present as a diff instead of a full re-draft: what
changed in the design, the proposed patch to the ticket, and any new/flagged
sub issues — not the entire ticket re-stated.

Followed by a short summary line: how many tickets, how many are net-new vs.
duplicates-avoided-or-updated, how many got sub issues and why, and any
granularity overlap flagged in Step 3.

## Rules

- Never invent frame content, copy, or behavior that isn't visible in what you
  were given — ask instead.
- Never double-count a shared component across multiple screen tickets.
- Never create or update anything in Linear before the human confirmation in
  Step 7.
- Always check for an existing issue before drafting a new one for the same
  screen/component/page.
- Always run [[content-model-review]] before drafting a ticket that defines
  a content type — never re-derive the content-model check inline, and never
  let two tickets define the same type under different names.
- Always check for dependencies beyond the page's own content (reused
  entries, links to other pages, integrations, missing assets) — don't
  leave them implicit.
- Always split QA into a content-parity subtask (files under the page's own
  milestone) and a design pixel-fidelity subtask (files under AVH22) rather
  than one generic QA line.
- Every ticket's milestone is a real AVH01–AVH22 proposal code, checked
  against the milestone reference — never a placeholder.
- Keep every ticket traceable to a specific Figma node, not just "the designs"
  generally.
- Always state each ticket's design-fidelity basis; never let a
  "pixel-perfect" acceptance criterion stand without saying whether it's
  backed by structured design data or by visual inspection only.
- Decompose into sub issues based on what a page actually contains, not a
  fixed template — and say explicitly when you considered it and skipped it.
- Never silently edit, reopen, or cancel a sub issue that's already Done or
  In Progress — flag it for the human instead.
