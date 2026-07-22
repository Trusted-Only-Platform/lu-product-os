# lu-product-os — Execution Protocol

This is the shared, platform-neutral build protocol for all of Lu's projects. Any coding agent (Claude Code, Codex, Devin, Factory, or a human) working in a repo that points here follows this document. Project-specific facts (commands, rules, progress) live in that repo's `CLAUDE.md` / `AGENTS.md` — this file holds everything that is identical across projects.

**The one rule that governs everything: gate outcomes, not activities.** Your work is done when the outcome gates below are green, not when a list of steps has been recited. Evidence must be a byproduct of doing the work (commits, CI runs, PR content, screenshots), never a report about it.

---

## Outcome Gates

A change merges when its tier's gates are green. Gates are enforced by machinery (CI + branch protection) — they cannot be skipped, waived by an agent, or satisfied by narration.

**Detection mode (free-plan repos):** where branch protection is unavailable (private repo on GitHub Free), merges cannot be physically blocked. The `main-guard` CI job then runs on every push to the default branch and raises a loud alarm (red run + labeled issue) on any merge that didn't come from a green PR. Doctor reports this as detection mode. The rules of conduct are identical — agents never merge, never push to main — detection mode just changes what happens when the rule is broken: caught within minutes instead of prevented.

| # | Gate | Meaning | Checked by | Tier |
|---|------|---------|-----------|------|
| G1 | **Correct** | Typecheck, lint, tests, build all pass on a clean checkout. Legacy repos with a pre-existing failure baseline: G1 = no NEW failures vs a committed, shrink-only baseline compared as sorted sets — encoded in the verify script, never improvised per session | CI runs `scripts/lu-product-os-verify` | All |
| G2 | **Intentional** | PR carries the planned acceptance criteria (blocking); UI changes carry screenshots (advisory — warns, never blocks) | `lu-product-os-pr-check` in CI | All |
| G3 | **Independently verified** | A non-builder agent verified the change at the depth pr-check computed (see Tiers) and posted `VERDICT: approve` | `lu-product-os-pr-check` in CI | Full |
| G4 | **Accepted** | Lu walked the preview deploy against the criteria | Only Lu can press merge (branch protection) | Full |
| G5 | **Hygienic** | Docs regenerated, ticket linked | Automated post-merge | All |

## Tiers

Rigor scales with blast radius. The tier is **computed mechanically** by `lu-product-os-pr-check` from the diff — an agent cannot argue its way into the light tier.

- **Light** (default): G1 + G2 + G5. Build → PR → CI green → merge.
- **Full**: adds G3 (independent verifier) + G4 (Lu's preview walkthrough), and a feature flag for user-visible changes on production-profile projects.

**Escalation to Full when ANY of:** counted diff > 300 lines (counts **exclude tests, lockfiles, snapshots, and docs** — diligence must not trigger escalation) · touches **critical paths** (migrations, auth, payments, webhooks, database schema — matched at path-segment boundaries, plus paths the repo marks sensitive in `.github/lu-product-os-sensitive-paths`) · touches **infra paths** (Dockerfile, docker-compose, `.github/`) · changes a **lockfile** (the mechanical signature of a real dependency change; a package.json script edit does not escalate). File count alone never escalates.

**G3 depth — pr-check computes it and posts it in the check summary:**

- `G3 depth: full` — critical-path or counted-line escalations. The verifier writes independent acceptance tests from the criteria, runs only those tests, reviews the diff, posts a verdict.
- `G3 depth: review-only` — infra or dependency escalations. The verifier reviews the criteria and diff hunks and posts a verdict. No test-writing: infra and dependency mistakes fail loudly at deploy/CI — independent tests add no protection there.

**Waivers:** only Lu can waive a gate, explicitly, in the PR (`WAIVE: G3 — <reason>`). Agents never waive gates.

## Project Profiles

Declared in the project's CLAUDE.md/AGENTS.md (`profile: prototype | standard | production`):

- **prototype** — git + `verify` script only. No PR ceremony, no verifier, no monitoring.
- **standard** — + GitHub PRs, CI, branch protection.
- **production** — + verifier, feature flags, tagged releases, rollback notes, error tracking, success metrics.

---

## The Build Loop

### Before you start (every session)
1. Read the project's `CLAUDE.md` / `AGENTS.md` — identity, commands, Key Rules, profile, verifier config, progress.
2. Read `0-plan/0-lessons.md` if present — apply its patterns.
3. Run `lu-product-os-doctor --quick` if available and state the machinery banner (`full / partial / detection mode / none`). If machinery is partial or absent, say so to Lu before proceeding; missing machinery means gates fall back to being your obligations. The banner also reports **codebase-docs freshness** from the Documentor's stamp — if docs read STALE, run Documentor Validate (and a targeted Update if needed) before building; if fresh, no pre-build doc check is needed.
4. Know your scope: a phase file (`0-plan/phases/`), a ticket brief, or a written scope statement. **Never edit code without a written scope.** If your scope came from a planning conversation, state it in the PR later. **Ticket handoff rule:** a ticket that carries neither acceptance criteria nor a pointer to a phase file is not executable — STOP and route it through Mode 2 planning; never write your own criteria for it.
5. Confirm your validation targets are reachable **before** choosing a validation path: local dev server starts, staging (if used) serves the right branch, auth-gated screens are reachable with available credentials. A validation plan that can't reach its target is a dead end discovered late.

### Re-entry (after compaction or interruption)
If you are resuming mid-build — after context compaction, a crashed session, or any gap where your memory of the work may be stale — reconcile against reality before acting: run `git log --oneline -15` and `git status`; compare commits against the task table or scope statement to see which tasks are already done; confirm the branch and whether a PR exists; restate the remaining scope in one or two lines, then continue. Never re-do work that commits prove is complete. **The repo is the truth; your memory is the hypothesis.**

### Build
6. **Branch** from the default branch: `phase-N/name` or `direct/short-description`. Never commit to main. **Push the branch immediately** — the push is your backup. (CI runs once the PR opens, not on every push.)
7. **Build task by task.** One commit per completed task, Conventional Commits format (`feat:` / `fix:` / `chore:` / `refactor:` / `test:` / `docs:`). Each commit leaves the code working. Push after every commit.
8. **Write unit tests alongside code** (your own feedback loop). The acceptance criteria from the plan are the definition of done; your tests must exercise them. **Local checks are the fast loop, CI is the gate:** while building, run only targeted checks (typecheck + the tests near what you changed). On **light tier**, do not run the full verify script locally — CI runs it once, on a clean checkout, at PR time. On **full tier**, run one clean `./scripts/lu-product-os-verify` before opening the PR, because a red CI discovered mid-verification wastes a verifier round.
9. **UI changes:** verify rendering in a real browser once per new or changed screen as you build (a screen that was never rendered is not done). Do NOT capture evidence screenshots per task — capture them once, at PR time, for the final state of each changed screen.
10. **E2E tests:** maintain one small, stable smoke suite for the app's critical paths. Extend it only when a change introduces a NEW critical path; never write per-phase E2E ceremony. The independent verifier owns criteria-coverage testing on full tier.
11. **Multi-repo workspaces:** scope names the affected repo(s); same-named branch + one PR per repo; merge in dependency order (backend first); check the workspace's cross-stack contracts doc before changing any shared surface.
12. **Lu's local review — conditional pause.** Pause and show the work on the local dev environment ONLY when direction risk is real: a new visual surface, genuinely ambiguous scope, or the first deliverable of a phase. Messy code is expected at that pause; Lu is judging direction, not quality. For bug fixes with a clear repro, backend work with unambiguous criteria, and small well-specified changes: do not pause — proceed to the PR, where criteria and screenshots are the review. **One human look per change:** if Lu reviewed locally, the preview walk is a quick spot check, not a second full review. Lu can always request a pause ("pause for my local review") or waive one ("skip local review" — note it in the PR body).

### Deliver
13. **Open a PR from the template.** The description must contain: what & why, the acceptance criteria (copied from the plan), screenshots for UI changes, deviations from plan, ticket reference (`fixes <ticket>`). The PR *is* the deliverable and the record — there is no separate report.
14. **Full tier: request independent verification — and run it in parallel with Lu's preview walk.** Read `verifier:` from CLAUDE.md/AGENTS.md and invoke it (MCP call or PR trigger comment such as `@codex review`), pointing it at the Verification Brief in the PR; the brief tells the verifier its depth (full or review-only) and its scope limits (no full-suite re-runs; re-reviews read only the delta). As soon as the verifier is invoked, if a preview URL exists, tell Lu once: the preview is up, verification runs in parallel, and he can walk it now or wait for full green. The two checks are independent — serializing them wastes the largest block of wall-clock time on full-tier changes.
15. **Fix until green.** Red CI or a request-changes verdict names what's missing — address it, push, let checks re-run. If Lu already walked the preview, tell him which areas changed so any re-walk is scoped. Do not ask Lu to override machinery.
16. **Stop.** Merging is Lu's action (squash merge). Do not merge, do not nag. If everything is green, say so once.

### After merge (automation + light touches)
17. **Codebase docs regenerate post-merge** (Documentor Update), which writes the freshness stamp (`0-documentation-codebase/.docs-fresh-sha`) doctor checks at session start — if the project has no automation for this, trigger the Update at the end of the session. If you changed business rules or domain concepts, update the product-intent docs in the same PR. **Flip the progress checkbox** in CLAUDE.md/AGENTS.md — a one-line inline edit to both files; that is the entire progress update.
18. **Prune verifier-authored tests:** keep a verifier's `test:` commits only where they cover a criterion or edge case the builder's tests don't; delete the redundant ones so the suite doesn't grow without bound.
19. Production promotes happen by tagged release (`vX.Y.Z`), decided by Lu. Risky changes ship dark behind a feature flag and ramp.
20. Add to `0-plan/0-lessons.md` only if something genuinely changed how future work should be done.

---

## Release Rules (production profile)

- **Migrations:** ship in the same PR as the code; CI runs them against a scratch DB; expand/contract pattern (add → dual-write → backfill → remove later); verified backup before any production migration; note the rollback per migration.
- **Rollback:** revert the squash commit / redeploy the previous tag. Each project's CLAUDE.md states its exact rollback command.
- **Watch window:** after a production promote, error rates are watched for 24–48h; new errors become tickets.

## Documentation Rules

- **Document only what code cannot say** (business rules, domain vocabulary, decisions and why). Everything code can say (schemas, API surfaces, env vars, module maps) lives in *generated* docs — regenerate, never hand-reconcile.
- **Never hardcode live infrastructure facts** (hosts, environment URLs, service lists) in prose docs — derive them at use-time (CLI, env vars) or verify them via doctor. If a fact can be derived, prose may name the command, never the value.
- History = git log + merged PRs. No manual change logs.
- The backlog is Linear. Every ticket states which of two kinds it is: **(a) a self-contained Mode B brief** — context, steps, acceptance criteria, cautions, executable directly; or **(b) a phase pointer** — references a phase file that Mode 2 planning already produced. A ticket that is neither is not executable (see the ticket handoff rule above).

## Principles

1. **Agents forget** → state lives in artifacts (repo, PR, ticket), never in conversation.
2. **Builders self-confirm** → verification is independent.
3. **Prompts don't enforce** → machinery does; if machinery is absent, announce it.
4. **Every artifact rots** → it must earn its existence or be generated.
5. **Lu's attention is the scarce resource** → he decides what to build, what "right" means, and whether to accept; everything else is yours. His touchpoints: approve the plan, ONE review look per change (the early local build when direction risk is real, otherwise the preview/PR), press merge.
