# OQTS org guidance for coding agents

Agent-facing operating rules for every `oqts/*` repository. Not human
onboarding: for that read the repo `README.md`, and the org
[`CONTRIBUTING.md`](CONTRIBUTING.md). This file is loaded at the workspace
root via a symlink; each repo carries its own `CLAUDE.md` for repo-specific
rules, which take precedence over this file where they conflict.

## Repo map

Checkouts sit side by side under one workspace root, so `../planning/X.md`
resolves.

| Repo | Visibility | Holds |
|---|---|---|
| `planning` | private | Cross-repo specs: architecture, VPS, data homes, OXDAQ plan |
| `oqts-design` | public | Brand system, tokens, logo assets. Source of truth for all styling |
| `oqts-site` | public | Public marketing site, Next.js on Vercel, oqts.org |
| `oqts-platform` | private | Back-office platform + API, VPS, Postgres. Identity spine |
| `oxdaq-core` | public | OXDAQ exchange engine, protocol, `oqts-client` SDK, dashboards |
| `oxdaq-ops` | private | OXDAQ bot implementations, round configs, admin tooling |
| `oqts-infra` | private | VPS compose files, Caddy config, deploy |
| `.github` | public | Org defaults, CONTRIBUTING, templates, this file |

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
