# OQTS org guidance for coding agents

Agent-facing rules for every `oqts/*` repository. Loaded at the workspace
root through a symlink. Humans start at each repo's `README.md` and the
org [`CONTRIBUTING.md`](CONTRIBUTING.md). Each repo's own `CLAUDE.md`
takes precedence over this file where they conflict.

## Repo map

Checkouts sit side by side under one workspace root. Read a repo's
`CLAUDE.md` before working in it.

| Repo | Vis | Stack | Status | Holds |
|---|---|---|---|---|
| `oxdaq-core` | public | Python 3.13, asyncio, FastAPI; C++20 engine | in build | The OXDAQ exchange, its `oqts-client` SDK, dashboards, and the OXDAQ spec in `docs/` |
| `oxdaq-ops` | private | Python, config | pre-build | Bot implementations and parameters, round configs, admin tooling |
| `oqts-infra` | private | Compose, Caddy | pre-build | The VPS: Caddy, compose files for every stack, runbooks, estate docs in `docs/` |
| `oqts-design` | public | CSS, SVG, build tools | live | Brand system and tokens. **Source of truth for all styling** |
| `oqts-platform` | private | FastAPI, Next.js, Postgres 16 | **live**, being replaced | Back office, public API, member identity. Holds applicant CVs. `docs/MIGRATION.md` hands it to the Django rewrite |
| `oqts-site` | public | Next.js on Vercel | **live**, being replaced | oqts.org. Folds into the Django rewrite |
| `.github` | public | Markdown | live | Org defaults, CONTRIBUTING, this file |

A teammate is rewriting `oqts-site` and `oqts-platform` as **one Django
repo**. Until its cutover, the two live repos stay authoritative for what
is deployed; change them only for live fixes, and record anything the
rewrite must carry in `oqts-platform/docs/MIGRATION.md`.

`planning` is archived. Its documents now live with what they govern.

## Where documents live

| Kind | Home |
|---|---|
| OXDAQ spec, protocol, build order, decision log | `oxdaq-core/docs/` |
| OXDAQ research ideas | `oxdaq-ops/docs/` |
| Web app product specs (platform plan, CV book, demographics), migration guide | `oqts-platform/docs/`, moving with the rewrite |
| Estate: architecture, VPS plan, data homes, Drive | `oqts-infra/docs/` |
| Brand | `oqts-design/brand.md` |
| GitHub org setup (historical) | `.github/docs/org-setup.md` |

## How the repos connect

- **One box.** The society VPS runs Caddy, the web app, Postgres and
  OXDAQ. `oqts-infra` will own Caddy and the compose files for all of
  them; until the Django cutover the live Caddyfile is in
  `oqts-platform/deploy/`.
- **The Caddy allow-list is a security boundary.** A new public API path
  that is not added to it 404s in production; a path added carelessly can
  publish admin routes and applicant CVs.
- **Member identity lives in the web app.** Its members table issues
  OXDAQ round tokens and, later, data-API keys. One list of humans.
- **OXDAQ: open engine, closed configuration.** `oxdaq-core` is public;
  `oxdaq-ops` is private because participants are meant to
  reverse-engineer the bots during a round, not before it.
- **Brand flows outward from `oqts-design`.** Consumers pin it; a change
  reaches a consumer only when it bumps its pin. No consumer defines
  styling of its own.
- **Competition exhaust becomes research data.** OXDAQ trade tapes are
  exported, later served to members through the web app.

## The working agreement (Alec's repos)

Alec leads the society's technology and builds OXDAQ to understand it end to
end, not to have it produced for him. That outranks delivery speed. The
Django web app is the teammate's; its repo sets its own rules.

### 1. Understanding outranks throughput

Prefer the slower build that leaves a better mental model. Assume the
work will be defended in front of sponsors and members.

### 2. Ownership boundary

| Alec writes by hand | Claude writes |
|---|---|
| Matching engine and order book, Python and C++, and the C++ bindings | Scaffolding, packaging, build systems, CI, tooling config |
| Cash and position accounting, validation | Message schema and codec |
| Price process maths | Tests, harnesses, fixtures, generators, benchmarks |
| Bot strategy logic | All frontend |
| All async: gateway, order loop, broadcast, the client library | Auth, tokens, sessions, deployment, security surfaces |

When work falls in Alec's column, do not write it, and do not sketch it
inline "to illustrate". Instead: say it is his; give the design (types,
invariants, failure modes); link the exact documentation, explaining any
non-obvious language or library feature without condescending; offer the
test that tells him when it is right. Stubs only when he asks, and they
raise `NotImplementedError`.

### 3. Design decisions route through Alec

Schema shape, module boundaries, data structures, dependencies, protocol
changes and participant-facing names constrain the future: present
options with a recommendation, then wait. Implementation inside an agreed
boundary needs no round trip.

### 4. Specs are binding

A repo's `docs/` spec is the contract for that repo. Do not deviate
without changing the spec in the same pull request. Flag contradictions
rather than resolving them silently.

### 5. Only as complicated as it needs to be

Clean, clearly laid out, current. Fewer and shorter documents; one place
for each fact; history in a decision log, not inline.

## House rules

- **Never commit secrets.** See `CONTRIBUTING.md`.
- **Branch and PR.** Never push to `main`. Branch `alec/short-description`,
  one logical change per PR, code-owner review required.
- **Commit messages:** short and imperative.
- **Single source of truth.** Styling in `oqts-design`, hierarchy in the
  GitHub org, the member roster in the web app's database. Never fork a
  copy.
- **No em-dashes** in anything a human reads on a surface we ship (UI,
  site text, docs, headings). Use a colon, a comma or a full stop.
- **Brand compliance** for anything with a UI: read
  `oqts-design/brand.md` first; tokens come from the design repo, never a
  hand-coded hex value.

## Documentation convention

Every repo has three files at its root, and specs in `docs/`:

- `README.md`: for humans. What it is, why, how to use it.
- `CLAUDE.md`: for agents. Rules, boundaries, invariants, commands,
  gotchas. Terse, no duplication of the README.
- `state.md`: where it stands, what was decided, what is next; about 20
  lines, updated when direction, results or next steps change.
