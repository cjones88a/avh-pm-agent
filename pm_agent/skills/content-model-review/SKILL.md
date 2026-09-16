---
name: content-model-review
description: Checks a page's content blocks against the current Contentful content model and decides, per block, whether it reuses an existing type, needs a new field on one, or genuinely needs a new type — then drafts specs for whatever's missing. Use when scoping what a page needs from Contentful, when figma-to-tickets or draft-ticket needs a content-model check before defining a type, or when the question is just "what does this page need from Contentful."
---

# Content Model Review

Before any ticket defines, migrates into, or modifies a Contentful content
type, this skill answers one question with evidence, not assumption: does
the type this page needs already exist, under this name or another one? A
content model drifts fast when several tickets each define their own
version of "the same" type under slightly different names — this skill is
the check that stops that.

## When to use this

- [[figma-to-tickets]] step 4/5 calls this before drafting any ticket that
  defines a content type — don't duplicate this logic inline there, run
  this skill instead.
- [[draft-ticket]] calls this for the same reason when an idea involves a
  content type.
- Run it directly whenever the ask is just "what does this page need from
  Contentful, and what already exists" — no new ticket involved yet.

## Step 1: Read the ticket or page

Get the concrete list of content blocks in scope — from the Linear ticket
if one exists, from the Figma frame, from a live-site audit
([[review-current-site]]), or from the human directly. Don't start
classifying until you have an actual list of blocks, not a vague sense of
"the page has some cards and a hero."

## Step 2: Establish the current content model — don't assume it

Check sources in this order, and say which one you actually used:

1. **The Contentful MCP connector, if available.** This is the primary path
   now that developers have Contentful MCP set up — read the space's actual
   content types and fields directly. This is the only source that tells
   you what's really built, not what was scoped.
2. **The maintained content-model reference doc**, if the connector isn't
   available or doesn't cover this space yet. Treat this as the best
   current specification, not as a system of record — it can drift from
   what's actually in Contentful.
3. **Prior Linear tickets that scoped or built related types**, as a last
   resort when neither of the above has an answer. Note explicitly that
   this is ticket archaeology, not a direct read of the model, since it's
   the least reliable source.

If the connector is available and disagrees with the reference doc, the
connector wins — flag the drift and correct the reference doc (Step 6)
rather than trusting the stale doc.

## Step 3: Classify each block

For every content block in scope, decide one of three outcomes:

- **Reuse as is** — an existing type already covers this block's fields
  exactly. Cite the type by name and where else it's used.
- **Reuse and extend** — an existing type is the right shape but is missing
  a field this block needs. Name the type and the field to add.
- **New type** — nothing existing covers this block's shape. Only reach
  this conclusion after actually checking Step 2's sources — never because
  a block merely *looks* new in a design tool. A "Stats Card" in Figma and
  a "Stat/Metric Callout" already in Contentful are the same failure this
  skill exists to catch.

Watch for near-duplicates: two blocks that look different in the design but
would produce the same content shape (e.g. a "3 most recent posts" teaser
and a "full post index" — both consuming the same underlying article entry
shape, just queried differently) should share one type, not each get their
own.

## Step 4: Draft the spec

For anything classified "reuse and extend" or "new type," draft a concrete
spec: type name, fields with types, which existing content it needs to
compose with or reference, and which pages will use it. Keep it implementable
without further guessing — a coding agent building this in Contentful
shouldn't have to invent field names or types.

## Step 5: Write back only after confirmation

Present the classification and any drafted specs to the human before
anything gets created:

1. Show the per-block classification and the reasoning behind each "new
   type" call specifically (since that's the expensive mistake to get
   wrong).
2. Ask explicitly whether it needs changes — silence is not approval.
3. Only after confirmation does this become a subtask (or ticket) in
   Linear, or an actual write against the Contentful MCP connector if the
   human has asked for the type to be created directly rather than just
   ticketed.

## Step 6: Update the reference doc

Whichever source in Step 2 turned out to be authoritative, update the
maintained content-model reference doc in the same pass: record the new or
extended type, note which pages use it, and correct any naming drift found
against the Contentful MCP connector. The next review should start from
this doc being current, not from re-deriving everything from ticket
history again.

## Output format

```
## Content model review: <page/ticket>

**Model source used:** Contentful MCP connector | reference doc | prior tickets (ticket archaeology)

### Block classification
- **<Block name>** — Reuse as is (<type>) | Reuse and extend (<type>, +<field>) | New type (<proposed name>)
  <one line of reasoning, esp. for "new type">

### Drafted specs (for anything not "reuse as is")
**<Type name>**
Fields: <name: type, ...>
Composes with / references: <other types, or "none">
Used on: <pages>

### Naming drift found
- <existing name> vs <name used elsewhere>, or "None found"

### Open questions
- <anything genuinely unresolved, or "None">
```

## Rules

- Never conclude "new type" without having actually checked Step 2's
  sources in order — a design tool showing something that looks new is not
  evidence it doesn't already exist.
- Never let two tickets define the same type under different names — if you
  find one already in flight, flag it rather than silently picking a name.
- Prefer the Contentful MCP connector over the reference doc over ticket
  archaeology, and say which one you used.
- Update the reference doc in the same pass a real decision gets made —
  don't leave it to drift further out of date.
- Nothing gets created in Linear or in Contentful before the human confirms
  the classification and specs.
