# Egeria Corporation

Free, open source tools for grant consultants, nonprofit fundraising consultants, and emerging
development professionals — for work that currently costs four figures a year.

Everything here is Apache 2.0. Sponsored by [OpenGrants](https://opengrants.io).

## The tools

| Repo | What it does | Run it | Hosted |
|---|---|---|---|
| [`grantcheck`](https://github.com/egeria-corporation/grantcheck) | Federal grant readiness by EIN — exemption status, Publication 78 deductibility, automatic revocation history, filing recency, SAM.gov registration and UEI, single-audit screen | `uvx grantcheck --ein 27-0125367` | `check.opengrants.io` |
| [`funder-graph`](https://github.com/egeria-corporation/funder-graph) | The open 990 funding graph — who funded whom, derived from Form 990-PF Part XV and Form 990 Schedule I, published as versioned Parquet | DuckDB against a URL | `funders.opengrants.io` |
| [`precedent`](https://github.com/egeria-corporation/precedent) | Federal award history and the pass-through finder, built on USAspending and single audit SEFA data | `uvx precedent history 93.243` | `awards.opengrants.io` |
| [`answerbank`](https://github.com/egeria-corporation/answerbank) | Local-first Markdown narrative library with staleness flagging and an MCP server | `uvx answerbank stale` | `answers.opengrants.io` |
| [`grantdesk`](https://github.com/egeria-corporation/grantdesk) | Pipeline, post-award compliance calendar, and win/loss ledger | `npx grantdesk demo` | `desk.opengrants.io` |

Each repo ships a **command-line tool** for people and an **MCP server** for agents, over the same
library. Each has a free hosted companion that adds no capability the tool lacks — it exists so
that someone who will never open a terminal can still get the answer, at a permanent URL.

## What these are built on

Public federal data, and the community work that made it tractable:

- **IRS** — Form 990 series bulk e-file XML, and the Tax Exempt Organization Search bulk downloads
  (Business Master File, Publication 78, Automatic Revocation List, Form 990-N)
- **[Nonprofit Open Data Collective](https://github.com/Nonprofit-Open-Data-Collective)** — the IRS
  E-file Master Concordance File, which is what makes 990 XML tractable across hundreds of schema
  versions, and the single most important upstream asset in this program
- **[GivingTuesday](https://990data.givingtuesday.org/tool-repository/)** — `form-990-xml-mapper`
  and `form-990-xml-parser`
- **USAspending**, the **Federal Audit Clearinghouse**, **SAM.gov**, and **Grants.gov**

Attribution here is a requirement rather than a courtesy. Every repo's `NOTICE` and
`docs/research/prior-art.md` name what it builds on, and fixes that belong upstream go upstream
first. We do not re-implement what a community project already does well.

## How these are built

Every repository follows the same conventions, published in this repo:

- **[`CONVENTIONS.md`](./CONVENTIONS.md)** — the two hard rules, the dual CLI + MCP interface, the
  optional-never-required OpenGrants integration, attribution, data honesty, and the required
  disclosure text
- **[`HOSTING.md`](./HOSTING.md)** — the hosted companion design, and why data-backed sites render
  at the edge rather than pre-render
- **[`DECISIONS.md`](./DECISIONS.md)** — the program decision record

Two rules govern all five:

1. **Easy to deploy and run.** One command, no account, no API key, no database to stand up. If a
   new user cannot get a real result inside 60 seconds of reading the README, the design is wrong.
2. **Simple, but solving a real problem.** Every repo does one job. Each carries a
   `docs/NON-GOALS.md` written before its first commit, so scope creep has something to argue with.

## On accuracy

These tools report facts from public data, with the source and publication date on every line. They
do not make eligibility determinations, predict outcomes, or give legal, tax, or accounting advice.

> This is informational only, derived from public data on the dates shown. It is not an eligibility
> determination, and not legal, tax, or accounting advice. Verify against the official source before
> relying on it.

## Contributing

Issues and pull requests are welcome on any repository. Each carries `CONTRIBUTING.md`,
`CODE_OF_CONDUCT.md` (Contributor Covenant 2.1), and `SECURITY.md` with private vulnerability
reporting enabled.
