# OQTS org guidance for coding agents

Agent-facing operating rules for every `oqts/*` repository. Not human
onboarding: for that read the repo `README.md`, and the org
[`CONTRIBUTING.md`](CONTRIBUTING.md). This file is loaded at the workspace
root via a symlink; each repo carries its own `CLAUDE.md` for repo-specific
rules, which take precedence over this file where they conflict.

## Repo map

Checkouts sit side by side under one workspace root, so `../planning/X.md`
resolves. **Every repo has its own `CLAUDE.md`**: read it before working
there. It takes precedence over this file where they conflict.

| Repo | Vis | Stack | Status | Holds |
|---|---|---|---|---|
| `planning` | private | Markdown | live | The binding cross-repo specs. `OXDAQ-PLAN.md` v2.0, `ARCHITECTURE.md`, `VPS-PLAN.md`, `DATA-HOMES.md`, `PLATFORM-PLAN.md` |
| `oqts-design` | public | CSS, SVG, Python/Node build tools | live | Brand system, tokens, logo and pattern assets. **Source of truth for all styling** |
| `oqts-site` | public | Next.js, TS, Vercel | live at oqts.org | Public marketing site. Forms proxy server-side to the platform API |
| `oqts-platform` | private | FastAPI, Next.js, Postgres 16, Docker | **deployed** on the VPS | Back-office platform and public API. **Identity spine.** Holds applicant CVs and personal data |
| `oxdaq-core` | public | Python 3.13, FastAPI, asyncio, uv | pre-build | OXDAQ exchange engine, protocol, `oqts-client` SDK, both dashboards |
| `oxdaq-ops` | private | Python, config | pre-build | OXDAQ bot implementations and tuned parameters, round configs, admin tooling |
| `oqts-infra` | private | Compose, Caddy, CI | pre-build | VPS compose files, Caddy config, deploy. Consumes `oxdaq-ops` as a submodule |
| `.github` | public | Markdown | live | Org defaults, CONTRIBUTING, templates, this file |

### How they connect

- **`oqts-design` flows outward.** `oqts-site`, `oqts-platform` and the
  OXDAQ dashboards each pin it as the submodule `vendor/oqts-design` and
  copy assets in with their own `scripts/sync-brand.mjs`. A brand change
  reaches a consumer only when that consumer bumps its pin. No consumer
  defines styling of its own, ever.
- **`oqts-platform` is the identity spine.** Its members table issues
  OXDAQ round tokens and data-API keys. One list of humans, three systems
  keyed off it.
- **A new public API endpoint is a three-repo change:** the route in
  `oqts-site`, the endpoint in `oqts-platform/signup-api`, and the path
  added to the Caddy allow-list. Miss the third and it 404s in
  production.
- **OXDAQ splits open engine from closed configuration.** `oxdaq-core` is
  public and holds the matching logic anyone may read. `oxdaq-ops` is
  private and holds the bot parameters participants are meant to
  reverse-engineer during a round, not before it.
- **Competition exhaust becomes research data.** OXDAQ trade tapes export
  into Postgres and files, which the platform's data API later serves to
  research teams.

## The working agreement

Alec is the sole engineer and the President of Technology. He is building
this to understand it end to end, not to have it produced for him. That
goal outranks delivery speed on every decision.

### 1. Understanding outranks throughput

Never optimise for lines shipped. If a shorter explanation and a slower
build produce a better mental model, take the slower build. Assume the
work will be defended in front of sponsors and members who will ask how
it works.

### 2. Ownership boundary

| Alec writes by hand, in neovim | Claude writes |
|---|---|
| Matching engine and order book | Repo scaffolding, packaging, CI, tooling config |
| Cash and position accounting | Message schema types and (de)serialisation |
| Price process maths | Test harnesses, fixtures, generators |
| Bot strategy logic | All frontend, without exception |
| All async: gateways, order loop, event dispatch, client library | All auth, tokens, session handling |
| | All user-facing and deployment code, security surfaces |

Rationale for the split: the concurrency model and the maths are the
system. Auth and deployment are where a hand-rolled version is a
liability, so Claude owns them for reliability and security. Frontend is
delegated entirely; Alec does not review it line by line, so it must be
correct and on brand without supervision.

When work falls in Alec's column, do not write it. Instead:

1. Say plainly that this one is his to write.
2. Give the design: the shape of the type or function, the invariants it
   must hold, the failure modes to guard.
3. Link the exact documentation he needs, including a short explanation
   of any non-obvious library or language feature involved (asyncio
   semantics, pandas/numpy idioms, `sortedcontainers` key behaviour,
   typing constructs). Do not assume prior familiarity; do not condescend.
4. Offer to write the test that will tell him when it is right. Tests are
   Claude's column.

Stubs are allowed in his column only when he asks for them explicitly,
and they must raise `NotImplementedError`, never contain a plausible
implementation.

### 3. Design decisions route through Alec

Anything that constrains the future is a design decision: schema shape,
module boundaries, data structures, dependency additions, protocol
changes, naming that will appear in participant-facing APIs. Present the
options with trade-offs and a recommendation, then wait. Do not present a
decision as settled because it is conventional.

Implementation detail inside an already-agreed boundary does not need a
round trip.

### 4. Specs are binding

`planning/` holds the authoritative specs. A spec says do not deviate
without updating the document first, and that is literal: change the
document in the same change set, or do not deviate. Flag contradictions
found in a spec rather than silently resolving them.

## House rules

- **Never commit secrets.** No exceptions. See `CONTRIBUTING.md`.
- **Branch and PR.** Never push to `main`. Branch as
  `alec/short-description`, one logical change per PR, code-owner review
  required.
- **Commit messages:** short, imperative. `Add order validation for short
  sells`, not `added stuff`.
- **Single source of truth.** Styling only in `oqts-design`. Society
  hierarchy only in the GitHub org. Member roster only in the platform DB.
  Never fork a copy.
- **No em-dashes in any copy that a human reads on a surface we ship**
  (UI microcopy, site text, docs, headings). Use a colon, a comma, or a
  full stop. House rule, brand.md 1.
- **Brand compliance is not optional** for anything with a UI. Read
  `oqts-design/brand.md` before writing a single style. Tokens come from
  the design repo via the submodule and sync-brand pattern; never
  hand-code a hex value.

## Documentation convention

- `README.md` is for humans: what this is, why it exists, how to use it.
  Prose, readable, public-facing tone.
- `CLAUDE.md` is for agents only: rules, boundaries, invariants,
  commands, gotchas. Terse and imperative. No marketing voice, no
  narrative, no duplication of the README.
- Every repo has both.
