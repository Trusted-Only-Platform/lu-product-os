# lu-product-os

The central machinery for Lu's product development process. Every project **inherits** its process from this repo instead of carrying its own copy — improve something here and every project gets it on the next CI run.

**Design principle: gate outcomes, not activities.** Agents (Claude Code, Codex, Devin, Factory) build however they like; the merge button only works when the outcome gates are green. Evidence is a byproduct of the work (commits, CI runs, PR content), never a report about it.

## What's in here

| File | What it is |
|------|-----------|
| [`lu-product-os-EXECUTION-PROTOCOL.md`](lu-product-os-EXECUTION-PROTOCOL.md) | The shared build protocol every agent follows — gates, tiers, build loop, release rules. Project AGENTS.md files point here. |
| [`.github/workflows/lu-product-os-ci.yml`](.github/workflows/lu-product-os-ci.yml) | THE reusable CI workflow. Projects call it with a ~15-line stub; it runs gate G1 (`verify`) and gate G2/G3 (`pr-check`). |
| [`scripts/lu-product-os-pr-check`](scripts/lu-product-os-pr-check) | Computes each PR's tier (light/full) mechanically from the diff and enforces: acceptance criteria present, screenshots for UI changes, verifier verdict on full-tier changes. |
| [`scripts/lu-product-os-doctor`](scripts/lu-product-os-doctor) | Idempotent **setup = check = repair**. Run in any project repo: scaffolds the verify script + CI caller + PR template, sets branch protection, reports the machinery banner. `--quick` for a read-only session-start check. |
| [`templates/`](templates/) | The pieces doctor installs into projects: CI caller, verify-script template, PR template (with the Verification Brief), AGENTS.md stub. |

## The gates

| # | Gate | Checked by | Tier |
|---|------|-----------|------|
| G1 | Correct — typecheck/lint/test/build pass | CI runs `scripts/lu-product-os-verify` | all |
| G2 | Intentional — acceptance criteria in PR; screenshots for UI | `pr-check` | all |
| G3 | Independently verified — non-builder agent posts `VERDICT: approve` | `pr-check` | full |
| G4 | Accepted — Lu walks the preview, Lu presses merge | branch protection | full |
| G5 | Hygienic — docs regenerated, ticket linked | post-merge automation | all |

**Tier escalates to full** (mechanically, in `pr-check`) when: counted diff > 300 lines · touches migrations/auth/payments/`.github`/Docker (path-segment matched) · changes a lockfile (real dependency change). File count alone never escalates. Extend per-repo via `.github/lu-product-os-sensitive-paths` (one regex per line).

**Detection mode (free plan):** private repos on GitHub Free can't have branch protection, so merges aren't blocked there. The `main-guard` job in the central workflow runs on every push to the default branch and raises a red run + labeled issue for any merge that didn't come from a green PR. Doctor reports this configuration as detection mode.

## Setting up a project

```bash
cd my-project
bash <(gh api repos/OWNER/lu-product-os/contents/scripts/lu-product-os-doctor --jq .content | base64 -d)
# — or, with this repo cloned locally —
~/Documents/lu-codebases/lu-product-os/scripts/lu-product-os-doctor
```

Doctor scaffolds/repairs everything it safely can, then tells you what it couldn't. Finish with a **canary PR**: open a trivial PR and confirm the checks run *and block*. Machinery isn't trusted until it has proven itself once.

Per-project footprint after setup (everything prefixed `lu-product-os-` so you can spot it):

```
my-project/
├── .github/workflows/lu-product-os-ci.yml    ← 15-line caller; real workflow lives here
├── .github/pull_request_template.md           ← installed from templates/ (GitHub requires this exact name)
├── scripts/lu-product-os-verify               ← the ONE project-specific piece: its quality gates
└── AGENTS.md / CLAUDE.md                      ← identity + profile: + verifier: + pointers
```

## One-time account setup (human)

1. `gh auth login` with a token that can administer your repos.
2. Decide this repo's visibility: **public** makes the reusable workflow and raw-script fetches work everywhere with zero extra config (it contains no secrets). If private: same-org repos must enable *Settings → Actions → Access → accessible from repositories in the organization*, and `pr-check` fetching needs a `LU_PRODUCT_OS_TOKEN` secret (see comments in the workflow).
3. Wire the verifier: install the platform's GitHub App (e.g., Codex) on the org and/or add its MCP server to agent config; put `verifier: …` in each project's AGENTS.md.
4. Optional, paid orgs: an org-level ruleset (PR required + required checks `lu-product-os / verify`, `lu-product-os / pr-check`) makes new repos born-protected; doctor then has nothing to repair.

## For agents landing in this repo

You maintain the process, not a product. Changes here affect **every** project on their next CI run — treat edits like production deploys: small PRs, and test workflow changes against a sandbox repo before merging. The protocol you follow while doing so is [`lu-product-os-EXECUTION-PROTOCOL.md`](lu-product-os-EXECUTION-PROTOCOL.md), same as everywhere else.

**The friction rule:** this OS changes only when a real project PR hits the same friction **twice**. No speculative process tuning — process iteration is itself a build-time sink, and untested "improvements" are how machinery rots. If you (or Lu) feel an urge to improve the process, log the friction instance somewhere findable and wait for the second occurrence.
