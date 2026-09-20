---
name: compare-products
description: Runs a rigorous, opinionated comparison of products, services, tools, or providers and writes the results as a set of markdown files (one per candidate, plus a README with a comparison table and a clear recommendation) — the same shape as a professional buyer's guide. Use this whenever the user wants to compare, shop for, choose between, or evaluate options for ANYTHING they'll buy, subscribe to, or adopt: physical products (hardware, appliances, vehicles), software/SaaS tools, service providers, vendors, contractors, or plans. Trigger on phrasing like "help me compare X", "which Y should I buy/get/use", "find me options for Z", "I need to choose between...", "recommend a...", or "/compare-products", even if the user doesn't use the word "compare" explicitly — e.g. "I need a new CRM", "what laptop should I get for video editing", "help me pick a payroll provider". Do NOT use for a single already-decided purchase with no comparison needed, or for comparing code/technical implementations (use code-review instead).
---

# Compare Products

Produce a buyer's-guide-quality comparison: real numbers, explicit tradeoffs, and an
actual recommendation — never a wishy-washy "it depends" with no opinion at the end.
This skill is domain-agnostic: it works the same way for a mini PC, a CRM tool, a
payroll provider, a car, or a coffee machine. Everything domain-specific happens
through the requirements you gather in step 1-2, not through special-cased logic.

## Why this shape

A comparison is only useful if someone can act on it later without re-deriving your
research. Writing one file per candidate plus a README that ties them together means:
the user can skim the README for the verdict, drill into one file when they want the
detail behind it, and you (or the user) can come back weeks later and update just the
piece that changed (a price, a new competitor, a changed requirement) without
rewriting everything. Treat the output folder as a living document, not a one-shot
report.

## Step 1: Understand what's being compared

Read the request for:

- **The category**: what kind of thing is this (hardware, SaaS tool, service
  provider, vehicle, appliance, subscription, contractor...)?
- **Hard requirements**: constraints that eliminate an option outright (must integrate
  with X, must fit in Y space, must support Z regulation, budget ceiling).
- **Nice-to-haves**: things that differentiate among otherwise-valid options but don't
  disqualify anything.
- **Scale/quantity**: how many units, seats, users, cameras, whatever the domain's unit
  of scale is — this often changes which tier of product makes sense.
- **Named candidates or links already in mind**: the user may paste product URLs,
  store search URLs, or brand/model names they're already considering. Treat these as
  must-include candidates and research them first — don't silently drop a candidate
  the user explicitly named, even if you'd recommend against it; include it with an
  honest verdict on why it's weaker.
- **Where/how they'll acquire it**: which store, region, deployment target, or
  timeline — this can change availability and pricing significantly (see Step 5).

## Step 2: Fill the gaps — but only the gaps that change the answer

If a genuinely decision-changing input is missing and can't be reasonably inferred
from context (an existing codebase, a prior conversation, an obvious default for this
domain), ask via `AskUserQuestion`. Group the questions into the natural "verticals"
for this domain — for a physical product that might be budget / technical
requirements / logistics-location / timeline; for a SaaS tool it might be budget /
must-have features / team size / integration constraints. Pick whatever verticals
actually apply; don't force a fixed template onto every domain.

Resist the urge to ask everything up front. A question is worth asking only if a
different answer would lead you to research or recommend differently. If the request
already pins down budget, scale, and hard requirements, skip the interview and start
researching — asking questions the user already answered reads as not having listened.
It's fine to do a short first pass, then ask a *second*, narrower round once research
surfaces a real fork in the road (e.g. "this pushes you toward a discrete GPU — do you
want that headroom or would you rather stay compact and cheaper?").

## Step 3: Research every candidate

Gather real specs, pricing, and independent opinions (reviews, comparisons, forum
consensus) for each candidate — both the ones the user named and the ones you find
that fit the requirements. Use whatever combination of these fits the domain:

- **WebSearch** for discovery: finding candidates you didn't know about, independent
  reviews, known issues, and comparisons other people have already written. This is
  usually your first move for an unfamiliar category.
- **WebFetch** for depth: reading a specific product page, spec sheet, pricing page,
  or a store link the user gave you. Expect some sites (large retailers especially) to
  block automated fetching (503s, CAPTCHA walls, bot detection) — when that happens,
  fall back to web search for the same information and say so rather than silently
  giving up on that source.
- **A specialized store or data-source tool, if one is available.** Before assuming
  you're limited to generic web search, run `ToolSearch` for something domain-relevant
  (a store search MCP, a pricing API, an internal catalog tool) — the user may have
  added one earlier in the session for exactly this purpose, and it often beats generic
  search for live price/stock/availability data. Don't hard-code the assumption that
  any specific tool exists, and don't block on it: if nothing relevant turns up, proceed
  with web search and mention once, briefly, that a specialized tool for this domain
  would sharpen the pricing/availability data if the user wants to add one (an MCP
  server, an API key, etc.) — don't belabor it.
- If a source's own tooling exposes something not readily generalizable as a normal
  tool call (e.g. a Python module meant to be used as an MCP server, but the MCP isn't
  registered for this session yet), it's reasonable to invoke its functions directly
  via Bash as a fallback, the way you'd use any other local script — just say plainly
  that's what you did.

As you research, keep track of what you couldn't verify. A spec you found on only one
low-confidence source, a price that's clearly stale, a review that's suspiciously thin
— note these so Step 5 can flag them rather than presenting them as fact.

## Step 4: Write the output

### Choosing the output location

**Ask before writing anything**, unless the location is already obvious and
unambiguous from context (see exceptions below). Comparison output is usually
personal research (a purchase, a vendor pick) rather than project source, so
don't default silently to the current working directory just because that's
where the session happens to be open — a shopping comparison landing inside
an unrelated git repo is exactly the kind of mistake this check prevents.

Use `AskUserQuestion` with (at least) these options, adapting labels to the
actual topic and paths:

- **A project-relative folder**, `<repo-root>/comparison/<topic>/` (not the
  bare working directory, and not the topic folder loose at repo root) —
  only offer this when a project is actually in scope (a git repo or clearly
  the active workspace), and always phrase it as a proposal to confirm, not
  a silent default: "save under `comparison/<topic>/` in this repo?"
- **A personal notes/vault location** if the user has one (check memory for
  a remembered vault path before asking them to type one from scratch). If
  that location holds more than just this one comparison, nest under a
  `comparisons/` subfolder there (e.g. `~/vault/comparisons/mini-pc-comparison/`)
  rather than dropping the topic folder at the vault's top level.
- Otherwise, let them name a path.

The point is confirmation, not silent choice either way: even the
project-relative `comparison/` option must be presented and agreed to, not
assumed, since a repo's working directory is not automatically the right
home for what is often personal purchase research.

Skip asking only when: the user already specified a path or folder name in
their request, or the comparison is unambiguously project-scoped (e.g.
comparing infrastructure/tooling choices *for* the repo the user is actively
working in, where the output is meant to inform that project's decisions and
likely be committed alongside it).

**Before creating a new folder, check whether one already exists for this
topic** (Step 6 covers re-invocation) — don't create `mini-pc-comparison-2`
next to `mini-pc-comparison` when the user is clearly continuing the same
comparison.

### One file per candidate

Each candidate gets its own markdown file, named in kebab-case after the
product/brand/model (e.g. `stripe.md`, `toyota-corolla-hybrid.md`,
`gmktec-nucbox-k9.md`). Structure each one as:

```markdown
# <Candidate name>

**TL;DR:** One or two sentences — what this is and the one-line verdict on it.

## Specs / details

A table of the concrete facts: specs for hardware, pricing tiers and included
features for software, terms and coverage for services, whatever the domain's
relevant facts are. Prefer numbers over adjectives.

## Fit for this [purpose]

A table or short list mapping the user's stated requirements to how well this
candidate satisfies each one. Be concrete: "handles 4 cameras comfortably" beats
"good performance." Call out anything genuinely disqualifying as clearly as anything
that's a strength.

## Availability / pricing

Where to get it, current price, stock/lead-time if relevant, and the date you checked
— this data goes stale fast and a reader needs to know how fresh it is.

## Sources

Links to everything you used: spec sheets, reviews, store pages. If you couldn't
fetch something (blocked, paywalled), say so here rather than silently omitting it.

## Verdict

Your actual opinion on this specific candidate: who should pick it, and what would
make you rule it out.
```

Adapt section names to the domain (a SaaS tool's "Availability / pricing" is just
"Pricing"; a service provider might need a "Contract terms" section instead of
"Specs") — the shape matters more than the exact headings.

### The README

The README is what ties the individual files together and is probably what the user
reads first, so it needs to stand on its own:

1. **Goal and constraints** — restate what you understood the user to want, including
   anything you asked about in Step 2. This lets the user immediately spot if you
   misunderstood something before reading further.
2. **A comparison table** — every candidate as a row, the requirements/key
   differentiators as columns. This is the "at a glance" view.
3. **A clear recommendation** — not a menu. Say what you'd actually buy/use, and why,
   in terms of the specific requirements from Step 1. Give one or two fallbacks for
   plausible ways the situation could differ (tighter budget, different priority,
   unavailable in their region) rather than hedging the primary recommendation itself.
   A comparison that ends in "it really depends on your needs" without committing to
   an answer has not done its job — you did the research, so have the opinion.
4. **Links to each candidate's file** for the reader who wants the detail.

Write the recommendation like you're advising a friend who's about to spend real
money or time: specific, reasoned, willing to be wrong but not wishy-washy.

## Step 5: Be honest about data quality

- Mark any spec or price you couldn't independently verify, and say what source it
  came from.
- Timestamp anything that changes on its own (price, stock, plan pricing, model
  availability) so a reader six months from now knows to double check.
- When a source blocked you (bot detection, paywall, login wall), say so plainly
  rather than pretending the information doesn't exist — "X's product page couldn't
  be fetched directly; specs below are from [other source]" is more useful than
  silently using a worse source without comment.
- Don't present a single forum comment or one enthusiastic review as settled fact —
  weigh it against other sources or flag it as anecdotal.

## Step 6: Handle re-invocation gracefully

The user may come back later to extend or correct a comparison rather than starting
fresh. Before treating this as a new comparison, check whether an output folder for
this topic already exists. If it does:

- **Adding a candidate**: research it the same way as Step 3, write its file, and
  update the README's table and (if it changes the answer) the recommendation. Don't
  rewrite files for candidates that haven't changed.
- **A new tool became available** (e.g. the user just added a store-search MCP):
  re-run the pricing/availability research for existing candidates using the new
  tool, and update just the availability sections and timestamps rather than
  regenerating every file from scratch.
- **Requirements changed**: re-read the existing files, figure out which candidates
  are affected, and update the fit assessments and recommendation accordingly. If the
  change is large enough that most of the comparison no longer applies, it's fine to
  say so and propose starting a fresh comparison instead of contorting the old one.
- Always read a file before editing it, and preserve the parts that are still
  accurate — the value of the living-document approach is lost if every re-invocation
  regenerates everything from scratch and discards prior refinement.

## A note on scope

This skill is for comparisons where research and judgment do the work — not for
comparing code, architectures, or technical implementations within a codebase (that's
a code-review or architecture-decision task). It's also not the right tool for a
single item the user has already decided on with nothing to compare against; if
there's truly only one candidate, a comparison folder isn't the right shape for that
answer — just answer directly.
