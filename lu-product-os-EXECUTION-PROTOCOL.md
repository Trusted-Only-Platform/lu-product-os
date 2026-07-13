# lu-product-os — Execution Protocol

This is the shared, platform-neutral build protocol for all of Lu's projects. Any coding agent (Claude Code, Codex, Devin, Factory, or a human) working in a repo that points here follows this document. Project-specific facts (commands, rules, progress) live in that repo's `CLAUDE.md` / `AGENTS.md` — this file holds everything that is identical across projects.

**The one rule that governs everything: gate outcomes, not activities.** Your work is done when the outcome gates below are green, not when a list of steps has been recited. Evidence must be a byproduct of doing the work (commits, CI runs, PR content, screenshots), never a report about it.

---

## Outcome Gates

A change merges when its tier's gates are green. Gates are enforced by machinery (CI + branch protection) — they cannot be skipped, waived by an agent, or satisfied by narration.

| # | Gate | Meaning | Checked by | Tier |
|---|------|---------|-----------|------|
| G1 | **Correct** | Typecheck, lint, tests, build all pass on a clean checkout | CI runs `scripts/lu-product-os-verify` | All |
| G2 | **Intentional** | PR carries the planned acceptance criteria; UI changes carry screenshots | `lu-product-os-pr-check` in CI | All |
| G3 | **Independently verified** | A non-builder agent wrote acceptance tests from the criteria, ran the suite, reviewed the diff, posted `VERDICT: approve` | `lu-product-os-pr-check` in CI | Full |
| G4 | **Accepted** | Lu walked the preview deploy against the criteria | Only Lu can press merge (branch protection) | Full |
| G5 | **Hygienic** | Docs regenerated, ticket linked | Automated post-merge | All |

## Tiers

Rigor scales with blast radius. The tier is **computed mechanically** by `lu-product-os-pr-check` from the diff — an agent cannot argue its way into the light tier.

- **Light** (default): G1 + G2 + G5. Build → PR → CI green → merge.
- **Full**: adds G3 (independent verifier) + G4 (Lu's preview walkthrough), and a feature flag for user-visible changes on production-profile projects.

**Escalation to Full when ANY of:** diff > 150 changed lines or > 5 files · touches migrations, auth, payments, or public API contract paths · adds or changes a dependency · the project CLAUDE.md marks the touched area as sensitive.

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
3. Run `lu-product-os-doctor --quick` if available and state the machinery banner (`full / partial / none — prompt mode`). If machinery is partial or absent, say so to Lu before proceeding; missing machinery means gates fall back to being your obligations.
4. Know your scope: a phase file (`0-plan/phases/`), a ticket brief, or a written scope statement. **Never edit code without a written scope.** If your scope came from a planning conversation, state it in the PR later.

### Build
5. **Branch** from the default branch: `phase-N/name` or `direct/short-description`. Never commit to main. **Push the branch immediately.**
6. **Build task by task.** One commit per completed task, Conventional Commits format (`feat:` / `fix:` / `chore:` / `refactor:` / `test:` / `docs:`). Each commit leaves the code working. **Push after every commit** — CI feedback arrives while you work.
7. **Write unit tests alongside code** (your own feedback loop). The acceptance criteria from the plan are the definition of done; your tests must exercise them.
8. **UI changes:** verify rendering yourself (browser tooling), capture screenshots — they go in the PR.
9. **Multi-repo workspaces:** scope names the affected repo(s); same-named branch + one PR per repo; merge in dependency order (backend first); check the workspace's cross-stack contracts doc before changing any shared surface.

### Deliver
10. **Open a PR from the template.** The description must contain: what & why, the acceptance criteria (copied from the plan), screenshots for UI changes, deviations from plan, ticket reference (`fixes <ticket>`). The PR *is* the deliverable and the record — there is no separate report.
11. **Full tier: request independent verification.** Read `verifier:` from CLAUDE.md/AGENTS.md and invoke it (MCP call or PR trigger comment such as `@codex review`), pointing it at the Verification Brief in the PR. The verifier writes acceptance tests from the criteria (without reading the builder's tests first), commits them as `test:` commits, runs the full suite, reviews the diff, and posts `VERDICT: approve` or `VERDICT: request-changes` with findings.
12. **Fix until green.** Red CI or a request-changes verdict names what's missing — address it, push, let checks re-run. Do not ask Lu to override machinery.
13. **Stop.** Merging is Lu's action (squash merge). Do not merge, do not nag. If everything is green, say so once.

### After merge (automation + light touches)
14. Docs regenerate on schedule (not your job mid-build). If you changed business rules or domain concepts, update the product-intent docs in the same PR.
15. Production promotes happen by tagged release (`vX.Y.Z`), decided by Lu. Risky changes ship dark behind a feature flag and ramp.
16. Add to `0-plan/0-lessons.md` only if something genuinely changed how future work should be done.

---

## Release Rules (production profile)

- **Migrations:** ship in the same PR as the code; CI runs them against a scratch DB; expand/contract pattern (add → dual-write → backfill → remove later); verified backup before any production migration; note the rollback per migration.
- **Rollback:** revert the squash commit / redeploy the previous tag. Each project's CLAUDE.md states its exact rollback command.
- **Watch window:** after a production promote, error rates are watched for 24–48h; new errors become tickets.

## Documentation Rules

- **Document only what code cannot say** (business rules, domain vocabulary, decisions and why). Everything code can say (schemas, API surfaces, env vars, module maps) lives in *generated* docs — regenerate, never hand-reconcile.
- History = git log + merged PRs. No manual change logs.
- The backlog is Linear. Tickets are written as self-contained agent briefs: context, steps, acceptance criteria, cautions.

## Principles

1. **Agents forget** → state lives in artifacts (repo, PR, ticket), never in conversation.
2. **Builders self-confirm** → verification is independent.
3. **Prompts don't enforce** → machinery does; if machinery is absent, announce it.
4. **Every artifact rots** → it must earn its existence or be generated.
5. **Lu's attention is the scarce resource** → he decides what to build, what "right" means, and whether to accept; everything else is yours.
