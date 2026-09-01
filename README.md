# egeria-corporation/.github

Org-level configuration and the program-level documents that every Egeria repository is built
against.

`profile/README.md` is what renders on
[the organization page](https://github.com/egeria-corporation).

## The binding documents

These are not background reading. Every build prompt in every repository names them as binding, and
each of the five repositories carries a vendored copy under `docs/program/` so that a fresh clone —
or a coding agent working in one — resolves them without fetching this repository.

| Document | What it governs |
|---|---|
| [`CONVENTIONS.md`](./CONVENTIONS.md) | The two hard rules, the dual CLI + MCP interface, optional-never-required OpenGrants integration, attribution, data honesty, the required disclosure, repo layout, engineering standards |
| [`HOSTING.md`](./HOSTING.md) | Cloudflare over Netlify and why, the site map, caching keyed on dataset vintage, and the SEO/GEO requirements that apply to every hosted site |
| [`DECISIONS.md`](./DECISIONS.md) | The program decision record. When a build prompt says to stop and ask, check here first — the question may already be answered |

## Templates

`ci-python.yml`, `ci-node.yml`, `gitignore-python`, and `gitignore-node` are the sources for each
repository's `.github/workflows/ci.yml` and `.gitignore`. They are already applied in all five
repositories; these copies exist so a sixth would start correctly.

## When a binding document changes

The vendored copies do not update themselves. Change the canonical copy here, then re-vendor into
each of the five repositories in the same change set, so that no repository is ever building against
a stale rule.

## Not published here

Two program documents are deliberately kept out of this public repository:

- **The competitive shortlist**, because it describes positioning in terms meant for internal use.
- **The research dossier's competitor pricing**, because several figures come from a third-party
  comparison rather than from a vendor's own pricing page. The program rule is that no competitor
  price appears in public copy until it has been re-verified on the vendor's own page and
  date-stamped. The vendored `docs/program/RESEARCH.md` in each repository carries everything else
  and withholds that section.
