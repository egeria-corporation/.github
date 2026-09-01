# Program Decision Record

Decisions made at the program level, binding on every repo. The build prompts each carry a
"stop and ask the human" section; when one of those questions is answered, the answer is recorded
here and the prompt is updated to point at the record rather than to block.

Format: what was decided, when, why, what it costs, and what would reopen it. A decision that
cannot say what would reopen it is a preference, not a decision.

---

## D-001 — Bundle the SAM.gov public entity snapshot into the published index

**Decided:** 2026-08-31 · **Status:** accepted · **Affects:** `grantcheck` core M5, `grantcheck`
hosted H4

### The decision

The `grantcheck` monthly ingest downloads the **SAM.gov Public Entity Extract**, derives a minimal
subset of public-tier fields, and publishes it inside the index shards that users download. The
keyless quickstart therefore covers all eleven checks, offline, with no account.

Fields taken, and nothing else:

```
UEI, legal business name, state, city,
registration status, registration expiration date, registration purpose
```

No Controlled Unclassified Information. No "For Official Use Only" tier. No sensitive tier — which
is where taxpayer identification number, banking information, and points of contact live. Those are
permanently out of scope and are listed in `grantcheck/docs/NON-GOALS.md`.

### Why

Three of `grantcheck`'s eleven checks come from SAM.gov: registration active, registration
expiration, and UEI presence. An expired registration is the most common avoidable disqualification
in the federal system, so dropping these checks materially weakens the tool.

The live API cannot carry them. GSA's published rate limits for the Entity Management API give a
**non-federal user with no SAM.gov role ten requests per day**; a user with a role, or a system
account, gets 1,000. Ten organization lookups a day is unusable for a consultant checking a client
roster, and it breaks the cron-over-a-roster workflow that `grantcheck`'s exit codes exist to
support. Verified against <https://open.gsa.gov/api/entity-api/> on 2026-08-31.

A bundled snapshot is therefore not a convenience. It is the only design in which the keyless
promise and the SAM checks coexist.

The alternative that preserves both without redistributing anything — a proxy endpoint on
`check.opengrants.io` holding our key — was **rejected outright**, and would stay rejected even if
the legal question came back the other way. It would make an open source tool depend on our
infrastructure to function, it would stop working the day we stop paying for it, and our logs would
record which EINs users research. That last one is forbidden by the core build prompt in as many
words: the tool must never report which EINs were checked.

### The risk we are accepting

The SAM.gov Public Entity Extract is, in GSA's own description, publicly available entity data
released under the Freedom of Information Act, and United States government works are not subject
to domestic copyright (17 U.S.C. §105). **Neither the Entity Management API nor the Entity Extracts
API documentation states any restriction on redistribution, republication, or reuse — and neither
grants permission either.** The risk is that silence, and it is procedural rather than substantive.

Two hedges, both running in parallel with the build rather than ahead of it:

1. **A written clarification request to GSA** via the Federal Service Desk, asking whether
   republication of a derived subset of the Public Entity Extract is permitted. Open before M5
   starts; do not wait on the reply to build.
2. **`grantcheck/data/SOURCES.md`**, published with the index, stating exactly which public-tier
   fields were taken, the extract date they came from, and that the artifact is a derived subset of
   a FOIA-releasable federal extract. A deliverable of M5, not a nice-to-have.

### What would reopen this

- GSA replies that republication is not permitted, or attaches conditions we cannot meet.
- GSA publishes redistribution terms that contradict the reading above.
- The extract begins carrying fields above the public tier, which would make the derivation step a
  filtering obligation rather than a convenience.

**The fallback, if reopened, is not the proxy.** It is to degrade the three SAM checks to `unknown`
unless the user supplies their own `SAM_API_KEY`, with the README saying so plainly. That path is a
configuration change rather than a rewrite, because both the CLI and the hosted site already have
to handle the low-confidence `unknown` case for organizations whose EIN cannot be matched to a SAM
entity with confidence.

### Unchanged by this decision

- **The EIN-to-UEI join is still inferred**, by normalized name and state, with a published
  confidence score and a `--uei` flag to pin it. Taxpayer identification number is sensitive-tier
  and is not searchable, so there is no lookup to be had at any tier we will use.
- **SAM.gov exclusions and debarment remain out of scope** on an inferred match. Accusing the wrong
  organization of being debarred is defamatory. Only a confirmed UEI, and even then only as a
  pointer to the official record.

---

## Resolved constraints

Not decisions, but facts that were open and are now closed. Recorded because several prompts plan
around them.

### C-001 — `opengrants.io` DNS is under our full control

**Confirmed:** 2026-08-31.

`_shared/HOSTING.md` and four of the five hosted build prompts treat the DNS zone as an unknown to
ask about on day one, on the grounds that it was the one launch step that could not be unblocked by
working harder. It is ours, and adding a CNAME plus a Cloudflare validation record is a task rather
than a coordination problem.

**Consequence for sequencing:** hosted launches are no longer gated on anything external. Each
hosted companion ships as soon as its repo is ready, so the program runs repo-then-site five times
rather than five repos followed by five sites.

---

## D-002 — No competitor naming or pricing in any repository

**Decided:** 2026-09-01 · **Status:** accepted · **Affects:** all five repos and all five hosted
sites

### The decision

No repository in this program names a commercial competitor or quotes its price — not in code, help
text, command output, documentation, a README, a build prompt, or a hosted page. The binding rule is
in `CONVENTIONS.md` under "No competitor naming or pricing."

Where the argument for a tool depends on the shape of the paid category, the category gets described
— "the paid foundation-research products," "subscription grant management software" — and the point
gets made without a name attached.

**This replaces the previous rule**, which permitted competitor pricing in public copy provided it
was re-verified on the vendor's own page and date-stamped. That rule created a standing verification
obligation before every publication, and the obligation is now gone along with the claims.

### What was removed

- All five `docs/research/competitive.md` files were rewritten as "What this replaces" — the same
  capability-gap argument with no vendor named and no price quoted.
- The commercial-product sections of `grantdesk` and `answerbank` `prior-art.md`. What those files
  keep is what the conventions actually require prior art to cover: the open source work each
  repository builds on, the incumbent practice it has to beat (the spreadsheet, the Google Docs
  folder), and the upstream contribution posture.
- Scattered comparative claims in `funder-graph`'s README and both prompts, `grantcheck`'s
  NON-GOALS, and `grantdesk`'s README.
- The competitor-price verification steps in the `precedent` and `answerbank` build prompts, which
  no longer describe anything that can happen.

The removed analysis is preserved outside the repositories, in the internal program folder. It was
archived before deletion rather than discarded.

### Three things the rule does not cover

1. **Grantmaker-side platforms named as domain facts.** A grantee files reports through a funder's
   portal, and a tool that models that reality names the portals — `grantdesk`'s `portal_kind` enum
   is a schema, not a comparison. These stay.
2. **Genuine attribution.** Where an organization maintained a form, standard, or dataset we build
   on, it gets credited by name. Attribution is a first-class requirement and outranks this rule.
3. **Our own costs and our sponsor's pricing.** Cloudflare figures and OpenGrants' published pricing
   are ours to state.

### Why

Durability rather than politeness. A named competitor claim is stale the moment their pricing page
changes, it invites a correction we then have to publish, and it makes a factual open-data project
read as a marketing exercise. The tools are more credible when they do the job and let the reader
draw the comparison.

The practical trigger: an audit before the first public push found roughly sixteen competitor price
figures across the five repos carrying a **VERIFY** marker, sourced from a third-party comparison
about two years old, plus one unqualified pricing claim in `funder-graph`'s most-read file.

### What would reopen this

A deliberate decision that a specific piece of public copy needs a named comparison to make sense —
a launch post, a conference talk. That is a communications decision, made once, for one artifact,
and it does not change what lives in the repositories.

---

## D-003 — Publish the index to Cloudflare R2, mirror to GitHub Releases

**Decided:** 2026-09-01 · **Status:** accepted · **Affects:** `grantcheck`, and the pattern every
later repo that publishes a dataset will follow

### The decision

The published index lives at **`https://index.check.opengrants.io`**, backed by the
`grantcheck-index` R2 bucket on our own Cloudflare zone. A GitHub Release is published from the same
build as the documented mirror.

### Why R2 rather than a metered host

Every user download is egress. R2 charges nothing for egress, so success is close to free; on a
metered platform the program working as designed becomes a bill, which is the failure mode where a
tool gets quietly throttled the moment it starts being useful. This is the reasoning already in
`HOSTING.md`; this decision is it being applied.

**Custom domain rather than the `r2.dev` subdomain.** Cloudflare rate-limits `r2.dev` and documents
it as unsuitable for production. The custom domain also means the URL is ours: if the storage ever
moves, the URL does not.

### Why PyPI is not part of this and cannot be

PyPI and Cloudflare are not alternatives. `uvx grantcheck` works because `uvx` resolves the name from
a Python package registry, and Cloudflare does not host one. The only alternative is
`uvx --from git+https://…`, which works but breaks the program's hard rule of one command and a real
result in under sixty seconds, and requires git.

With Trusted Publishing there is no token to store or rotate, so PyPI is in practice the
lowest-management component in the stack.

### Published layout

```
index.check.opengrants.io/manifest.json              the "latest" pointer, 5 minute cache
index.check.opengrants.io/{vintage}/manifest.json    the pinned copy
index.check.opengrants.io/{vintage}/shard-{NN}...    immutable, 1 year cache
```

Shards upload first and the manifest last, because the manifest is the commit point: until it
changes, clients keep resolving the previous vintage consistently, so a half-finished upload cannot
leave anyone reading a partial index.

The client resolves both the R2 layout and the flat release-mirror layout without configuration,
because GitHub release asset names cannot contain a slash.

### What this cost us to learn

Cloudflare returns **403 to the default `Python-urllib` User-Agent**. The upload succeeded and the
workflow's own verification step then failed fetching what it had just published. The descriptive
User-Agent in the client is therefore load-bearing rather than decorative, and there are now tests
pinning it.

### What would reopen this

- R2 pricing changing such that egress is no longer free.
- Download volume making the Cloudflare cache in front of the bucket insufficient, which would be a
  good problem and a tuning exercise rather than a change of host.
