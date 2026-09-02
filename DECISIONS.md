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

---

## D-004 — Free reports, gated workflow

**Date:** 2026-09-01 · **Status:** implemented (grantcheck) · **Decided by:** the user

Per-EIN reports, the check explainers and the JSON API are open, uncredentialed and crawlable.
An email address unlocks bulk check, saved rosters with monthly monitoring, and export.

### Why this split and not another

The reason to build a hosted companion at all is that a permanent, citable page per organization
is worth indexing and quoting. A login wall destroys exactly that property: a page a crawler
cannot read is a page no model can cite, and the argument for the site collapses. So the rule is
narrower than "gate the good stuff" — **signing in must never be the price of an answer.**

What is behind the gate is what genuinely needs to remember something across visits. Monitoring
cannot notify without an address. A roster cannot persist without an identity. Export is only
meaningful over a saved set. None of the three has any search value, so nothing is lost by
closing them.

### The privacy exception this creates, stated plainly

The architecture forbids profiling who looked up which EIN. A saved roster is precisely such a
join, and it is a deliberate, scoped exception:

- It exists only for accounts, only for organizations the account deliberately saved, and only
  because monitoring cannot work otherwise.
- Report pages remain unassociated with any account, signed in or not. They are served from cache
  and no per-account read is recorded.
- Deleting an account removes the roster, sessions and pending links immediately, with no email
  and no waiting. Leaving has to be real or the exception is not scoped, it is just a promise.

### Magic links, not passwords

Chosen by the user. There is no password to store, hash, rotate, reset or leak, and no
password-reset flow — the flow that most often goes wrong. The database holds a SHA-256 of each
token and never the token, so reading it does not hand out live sessions. Links are single-use
and expire in fifteen minutes.

Requesting a link creates no account: the pending email and name live on the token row, and an
account exists only once somebody has proved they can read that inbox. That is not incidental —
it is what lets the sign-in email honestly tell an unintended recipient that nothing was created
for them.

### What would reopen this

- Evidence that the free tier is being scraped in a way that costs real money, which would argue
  for rate limits on the JSON API rather than for gating pages.
- A monitoring volume that outgrows a single scheduled Worker invocation.

---

## D-005 — The 00 EIN prefix is real

**Date:** 2026-09-01 · **Status:** implemented · **Supersedes:** an assumption, not a decision

Both implementations rejected every EIN beginning `00`, with the comment "the IRS does not issue
prefix 00". **That is false**, and the IRS's own published files disprove it.

The August 2026 index carries **136 organizations with prefix 00** — 19 in the Business Master
File, 14 listed in Publication 78, and **90 on the automatic revocation list**. They are mostly
churches and small religious organizations.

The 90 are why this mattered. For an organization whose exemption is genuinely revoked — the
hardest of the hard stops this tool exists to find — grantcheck answered *"not a valid EIN"*. A
caller reads that as a typo and moves on. It converted a blocking finding into a silent one,
which is the worst failure mode available to a readiness check.

Now only the all-zeros placeholder is rejected. `00-0000000` still fails on format and still
never reaches the network.

### The general lesson, which is the point of writing this down

Ten EIN prefixes genuinely were never issued (07, 09, 17, 18, 19, 28, 29, 49, 79, 89) — that was
established by counting the real data. Prefix 00 was rejected on an assumption that *sounded*
like the same kind of fact and was never checked against the files. **Validation rules that
exclude real records are worse than no validation**, because they fail closed and silently, and
the caller cannot tell an invalid input from an unwelcome one.

Every remaining validation rule in either implementation should be traceable to a count over the
published data, not to plausibility.

---

## D-006 — Resolve by XPath, not by the concordance's `versions` column

**Date:** 2026-09-02 · **Status:** implemented (funder-graph M1) · **Supersedes:** an assumption in the build prompt

The funder-graph build prompt's stop-and-ask #1 says: if the concordance covers less than ~95%
of filings by volume, stop — the project becomes a concordance-contribution project first. The
first measurement against real filings looked like exactly that, and was not.

### What the data said

- **Form 990-PF is not in `concordance.csv`.** That file (6,864 rows) carries the core 990 and
  every schedule and has zero 990-PF rows. Part XV lives in a separate file,
  `02-concordance-foundations/F990-PF-FULL.CSV`. A loader reading only the main file reports 0%
  coverage of the primary edge list. It is a file-layout fact, not a coverage fact.
- **The `versions` column is stale; the XPaths are not.** Annotations for the Part XV and
  Schedule I subtrees stop at `2016v3.0` / `2018v3.x`. The XPaths flagged `current_version = T`
  match 2019–2022 filings exactly: every required Part XV field resolved on real filings at
  `2020v4.0`, `2021v4.2` and `2022v5.0`, and four Part XV rows summed to the filer's own stated
  total with delta zero. Schedule I resolved 12 of 17 fields on a `2021v4.2` filing, the five
  misses being optional leaves genuinely absent from it — and every Schedule I row carries an
  *empty* `current_version`, so "not flagged" cannot mean "not current" there.
- **Upstream already holds the per-version truth.** `03-versions/raw-mappings/` has one XPath
  inventory per schema version, `2016v3.0` through `2022v5.0`, each listing the 18 Part XV and
  20 Schedule I XPaths. The `versions` column was never regenerated from them.

### The decision

Field resolution selects the concordance's current XPaths and **does not gate on `versions`**.
A missing annotation is an upstream metadata gap to report, never evidence a field is unmapped.
Per-version presence is measured against `raw-mappings/` instead — the coverage matrix is a
join, not a hand-check — and the report distinguishes "resolved by XPath" from "annotated
upstream" so the gap stays visible and honest.

The contribution we owe upstream is therefore a PR that extends `versions` from their own
inventories, weighted by real filing volume, not a complaint that coverage is missing.

### Corrections to the build prompt's operating assumptions, all verified 2026-09-01

- The IRS year directory listing returns 404; enumerate the landing page's hrefs instead.
- There is no per-filing URL anywhere — IRS, the old S3 mirror, or ProPublica. `fetch-raw`
  must stream one member out of its ZIP.
- `SUB_DATE` is year-only in `index_2023.csv`. The index carries no ZIP column.
- `RecipientPersonNm` is routinely populated with organization names and, separately, appears
  in `ApplicationSubmissionInfoGrp` for the application contact. Neither may become a grant row
  to a person without the spec's "no organizational tokens" clause and a group-scoped read.
- Attachment-filed Part XV has a second shape the empty-group check misses: one placeholder row
  reading `VARIOUS ORGANIZATIONS / SEE ATTACHED SCHEDULE`, here for $9.76M.

### The general lesson

A measurement that says "0% coverage" deserves one more hour before it becomes a strategy
decision. Both times it appeared here it was an artifact — a second file, a stale column — and
the fix was to read the source layout, not to change the project's shape.

## D-007 — Entity resolution: Metaphone for the phonetic block, DuckDB for Jaro-Winkler, resolution as its own stage

**Date:** 2026-09-02. **Repos:** funder-graph. **Status:** decided.

The build spec (section 7) names double metaphone of the first two name tokens as the fourth
blocking key. funder-graph uses `jellyfish`'s Metaphone instead. The key only has to *recall*
candidates - the score decides - and jellyfish is maintained (Rust wheels, released 2025-10),
where every double-metaphone package on PyPI last released in 2016. If a measured recall gap on
the labeled set is ever attributed to the phonetic block, that is the moment to revisit, not
before.

Jaro-Winkler is DuckDB's own `jaro_winkler_similarity`, computed inside the blocking query
against both the legal and the sort name and prefiltered at the probable floor, so Python only
ever scores plausible candidates. `JW_PROBABLE = 0.90` is our floor for the tier-D guess; the
spec fixes only the tier-C threshold (0.94 with ZIP5 agreement).

Resolution is a separate stage (`build resolve`), not a step inside normalize. The spec's
"resolve each distinct tuple once" means the distinct
`(name_normalized, name_raw, city, state, zip5, ein_reported, recipient_type)` tuples are read
out of the written Parquet, resolved against `bmf`, remembered in a `resolutions` table in the
build state, and the partitions rewritten through one join that preserves row order and count.
Section 8's monthly update ("re-resolve last month's unresolved, never churn A/B") falls out of
that table for free.

`recipient_ein_source` gains one value, `manual_correction`, for rows assigned from
`data/overrides/ein-corrections.csv`. A hand-verified correction is not a BMF deterministic
match and must not be published as one. README and CHANGELOG record the addition.

### What is not decided here

The labeled evaluation set (>= 1,000 hand-verified pairs) and the per-tier precision gates are
an M4 exit criterion, not a design choice; they require the real corpus and the real BMF, and
"hand-verified" means a person checked them.

## D-008 — A coverage gate measures mapping, not data completeness

**Date:** 2026-09-02. **Repos:** funder-graph. **Status:** decided.

The first `build map` over the 2023 posting (469,093 grant-bearing filings) read **93.36%** and
stopped the build at the 95% gate, as the build spec requires. Read as a mapping failure it
was the third artifact of its kind in this project (see D-006). The evidence:

- Every one of the 23 XPaths present in filings that the pipeline does not consume is a
  990-PF `ApplicationSubmissionInfoGrp` contact field or a manager name - fields NON-GOALS
  says must never be published. No Schedule I path was unmapped.
- Every required field resolved on every schema version for both forms; 990-PF was at
  99.6-100% everywhere. The shortfall was Schedule I only, and per field it was
  `recipient_ein` (resolved in 84.7% of grant-bearing 2022v5.0 filings) and `purpose`
  (89.7%), while the recipient name resolved in 99.3%. The same XPaths that "failed" in
  15% of filings resolved in the other 85%: the mapping was right and the filers left the
  field blank.
- The EIN-less grantees are domestic: 2 foreign rows in 19,000 sampled, no individuals.
  Resolving them is what the matcher's tiers B-D exist for.

**Decision.** The gate requires the fields a row cannot be published without - a recipient
name and an amount (cash or non-cash on Schedule I) - which is what the coverage module's own
comment had always said. Purpose, the reported EIN and the address are tallied and published
as per-field presence, and the strict share is published beside the gate as
`all_common_fields_pct` so the original number stays visible rather than disappearing into a
redefinition. Corrected result: **99.43%** by volume, every version above 95%; presence
93.54%.

**The rule.** A gate must measure what its name says. "Coverage" is whether the concordance
maps the fields we read; "completeness" is whether filers filled them. Conflating them stops
the build on filer behavior and, worse, would tempt someone to loosen the target. When a gate
fails, the first hour goes to checking which of the two it measured.

**Not changed.** The 95% threshold, the stop-and-ask, and the reconciliation gate that follows.
