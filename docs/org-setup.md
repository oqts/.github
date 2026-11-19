# OQTS GitHub organisation: setup runbook

> Historical record of the org's bootstrap (summer 2026), moved from the
> archived `planning` repo on 2026-11-19 and kept verbatim. Current repo
> layout and rules: `../CLAUDE.md` and `../CONTRIBUTING.md`.

**Audience:** Alec + 4 core members bootstrapping the society's foundational infrastructure.
**Scope:** Full org configuration, repo layout, teams/permissions, protection, secrets, and the growth path for research/project avenues added later.

---

## 0. Design principles (what changed and why)

Three principles drive every decision below:

1. **The read-access boundary is a repo boundary, not a folder.** GitHub grants read access per repository, so anything confidential is its own private repo. This is unchanged.
2. **Model current reality, document the growth path.** Right now you have 5 equal, trusted builders. The earlier multi-tier contributor model was premature — all 5 are core. So: one `core` team today; the research/project tiers are *designed but not built* until you need them.
3. **Lock governance at the org level now so growth is cheap later.** Default permissions, repo-creation policy, and repo classification are set once, up front. Get these right and adding 20 research repos later costs you nothing in access-control rework.

---

## 1. Org-level configuration

Do these once, before inviting anyone.

| Setting | Value | Why |
|---|---|---|
| **Org owners** | Exactly 2 (you + one trusted co-founder) | Bus factor. A sole owner orphans the org if you lose access. Five owners is too many — owners can delete the org and remove each other. |
| **Base permissions** (Settings → Member privileges) | **None** | Default is "Read", which leaks the full repo list to every member. "None" means people only see repos their team explicitly grants — essential once researchers join. |
| **Two-factor auth** (Settings → Authentication security) | **Require 2FA for everyone** | Cheap, non-negotiable for an org holding deploy secrets. |
| **Repository creation** (Member privileges) | Restricted to owners (or a small maintainers set) | Keeps the flat repo namespace clean as it scales. Leads request a repo via an issue template (see §8). |
| **Default branch name** | `main` | Consistency. |
| **Billing** | Team plan, *paid by the society account* — or chase the education/nonprofit discount first | Required for branch protection on the two **private** repos. Public repos get it free. Do not bill this to a personal card (see Footguns). |

**License decision (do before publishing `oxdaq-core`):** pick one permissive license deliberately, since the engine is public and recruiter-facing. `MIT` (simplest, most permissive, recruiter-friendly) or `Apache-2.0` (adds an explicit patent grant). Either is fine; just choose on purpose, not by accident.

---

## 2. Repository layout

Four repos for the foundational infra, plus one org-config repo.

| Repo | Visibility | Contents |
|---|---|---|
| `oxdaq-core` | **public** | Matching engine, WebSocket protocol, order book, `BaseAlgorithm` SDK. Publishes `oqts-client` to PyPI. Keep isolated — it's a versioned, published artifact. |
| `oqts-platform` | **public** (see flag) | Society website + research data-access frontend/API. |
| `oxdaq-ops` | **private** | Bot implementations/params, round configs, admin tooling. Submodule into `oqts-infra`. |
| `oqts-infra` | **private** | Deployment, VPS provisioning/IaC, CI config, secret *templates* (not secrets). |
| `.github` | public | Org-wide defaults: profile README, issue/PR templates, `CONTRIBUTING.md`, default workflows, the "request a repo" template. |

**Not in version control:** research datasets (object store / DB with their own ACLs) and live secrets (secrets manager / Actions secrets). Never in any repo.

**`oqts-platform` visibility flag — your one open decision:** keep it public *only if* the data layer exposes nothing sensitive. If the research data carries licensing/redistribution terms or any personal data (GDPR — UK/Oxford), split the data-access API into a private `oqts-data` repo and keep only the marketing site public. Until you've confirmed, default the data API to **private**.

**Classification for scale:** define a repository **custom property** `tier` with values `core | research | project` (Org Settings → Repository → Custom properties). Tag the four infra repos `tier=core`. Later, org rulesets and access reviews key off the property instead of brittle name-matching. (Verify ruleset-by-property targeting is available on your plan tier; if not, fall back to a `research-*` / `proj-*` naming convention.)

---

## 3. Teams and access (current state: 5 core members)

One team today. Two of the five hold the owner role on top of team membership.

**`@oqts/core`** — all 5 members.

| Repo | `@oqts/core` | Owners (you + 1) |
|---|---|---|
| `oxdaq-core` | maintain | admin |
| `oqts-platform` | maintain | admin |
| `oxdaq-ops` | maintain | admin |
| `oqts-infra` | maintain | admin |

`maintain` lets the team manage repo settings and merge without handing out `admin` (which controls deletion, visibility, and access). During the foundational build, all 5 having access to the private repos is correct — they *are* the trusted inner circle. The read boundary that matters is between `core` and the **future** research/project teams, which never get added to these four repos.

---

## 4. Branch protection and CODEOWNERS

Apply to `main` on every repo (per-repo Rules, or an org ruleset targeting `tier=core`):

- Require a pull request before merging
- Require **1** approving review (not 2 — too heavy for a 5-person team bootstrapping; revisit for `oxdaq-ops`/`oqts-infra` once the team grows)
- Require review from Code Owners
- Require status checks to pass (your CI)
- Block force-pushes and branch deletion

Add a `CODEOWNERS` file to each repo so the right person is auto-requested:

```
# oxdaq-core/.github/CODEOWNERS
*                 @oqts/core
/matching/        @your-handle          # protocol-critical paths get a named owner
/protocol/        @your-handle
```

Reminder: branch protection on the two **public** repos is free; on the two **private** repos it requires the Team plan.

---

## 5. Secrets, environments, submodule wiring

**Secrets:** store deploy credentials as **org-level Actions secrets scoped to `oqts-infra` only** (Org Settings → Secrets → Actions), not repo-level copies scattered around. Nothing sensitive ever sits in a repo file or gets echoed into a build log.

**Production environment:** in `oqts-infra`, create a `production` environment with **required reviewers**. A deploy during a live OXDAQ round then needs an explicit human approval — your blast-radius control for the most downtime-sensitive moment. (Environment protection rules on a private repo need Team.)

**Private submodule in CI (real gotcha):** `oqts-infra` pulls `oxdaq-ops` as a submodule. A default `actions/checkout` *cannot* read the private submodule. Fix: add a **deploy key** (read-only) for `oxdaq-ops` and pass it to checkout:

```yaml
- uses: actions/checkout@v4
  with:
    submodules: recursive
    ssh-key: ${{ secrets.OXDAQ_OPS_DEPLOY_KEY }}
```

Use a dedicated deploy key, not a personal token — same succession logic as the billing.

---

## 6. Per-repo bootstrap checklist

For each of the four infra repos:

- [ ] `README.md` — what it is, how to run, where it sits in the system
- [ ] `LICENSE` — public repos only (MIT or Apache-2.0)
- [ ] `.gitignore`
- [ ] `CODEOWNERS`
- [ ] CI workflow stub (lint + test)
- [ ] Branch protection on `main`
- [ ] `tier` custom property set
- [ ] Team access granted per §3

For `.github`: profile README, `CONTRIBUTING.md`, issue/PR templates, and a **"Request a new repo"** issue form (used by the research-avenue workflow below).

---

## 7. Setup order

1. Create the org (github.com/organizations/new). Assign the 2nd owner immediately.
2. Apply all §1 org settings (base permission **None** first — before anyone joins).
3. Create the `.github` repo; add templates and the repo-request form.
4. Define the `tier` custom property.
5. Create the four infra repos; set visibility and `tier=core`.
6. Create `@oqts/core`; grant access per §3.
7. Invite the 5 members; require 2FA; add them to `@oqts/core`.
8. Run the bootstrap checklist (§6) per repo.
9. Wire secrets, the `production` environment, and the submodule deploy key (§5).
10. Confirm: a `core` member can open a PR on a private repo and *cannot* merge without a review.

Useful `gh` snippets (org must exist first):

```bash
# repos
gh repo create oqts-org/oxdaq-core    --public
gh repo create oqts-org/oqts-platform --public      # or --private per the flag
gh repo create oqts-org/oxdaq-ops     --private
gh repo create oqts-org/oqts-infra    --private
gh repo create oqts-org/.github       --public

# team + access (repeat the access line per repo)
gh api orgs/oqts-org/teams -f name=core -f privacy=closed
gh api -X PUT orgs/oqts-org/teams/core/repos/oqts-org/oxdaq-core -f permission=maintain

# submodule
cd oqts-infra && git submodule add git@github.com:oqts-org/oxdaq-ops.git ops
```

Branch protection payloads are verbose — set those via the UI (Settings → Rules) or an org ruleset rather than raw `gh api`.

---

## 8. Growth path: research and project avenues (later)

This is the part the up-front governance pays off. When you open the org to research/project ideas, **nothing about the core-infra access changes.** You add a parallel structure:

**Repos:** new repos are `tier=research` or `tier=project` (and/or `research-*` / `proj-*` named). Default them **public** — most research benefits from being open, and public repos keep cost and access simple.

**Teams:** one team per project, e.g. `@oqts/proj-vol-surface`. The project lead gets `maintain` on *their* repo; members get `write`. The `core` team is **not** added — that boundary is the whole point.

**People and cost control:** GitHub Team bills per *org member*, so to avoid every researcher consuming a paid seat:
- Keep their repos **public** and add researchers as **outside collaborators** (free on public repos), *or*
- Only make someone a full org member when they genuinely need private or cross-repo access.

Because base permission is **None** (§1), a researcher added to one project repo sees *only* that repo — never the infra, never other projects.

**Repo creation stays gated:** a lead opens the "Request a new repo" issue in `.github`; an owner creates it with the right `tier`, team, and protection in one go. This keeps the namespace governed as it scales from 5 people to many.

**Optional commons:** if you want all researchers to share a read-only "how to use the data infra / templates" repo, make a public `oqts-commons` repo and a parent `@oqts/community` team whose children (the project teams) inherit read on it.

---

## Footguns (read before you start)

- **Sole owner.** Orphans the org. → 2 owners.
- **Personal-card billing.** Lapses at a renewal you no longer control after you graduate. → society account.
- **Base permission left at "Read".** Leaks every repo to every member, breaking the research-avenue isolation before you've even built it. → set to **None** first.
- **Private submodule in CI without a deploy key.** Silent checkout failure. → dedicated read-only deploy key.
- **Secrets in repo files or build logs.** The classic "public website repo leaks a private credential." → org secrets scoped to `oqts-infra`; never print them.
- **Renaming `oxdaq-core` after PyPI publish.** Breaks installs and links. → name it right once.
- **Adding `core` to research repos by habit.** Erodes the boundary you set up. → resist; research teams are separate.
