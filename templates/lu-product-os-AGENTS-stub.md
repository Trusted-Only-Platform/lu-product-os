<!-- lu-product-os AGENTS.md / CLAUDE.md stub for a project repo.
     Copy as AGENTS.md (and/or CLAUDE.md) to the project root and fill the TODOs.
     Keep it under ~40 lines: identity, commands, config, pointers, project rules.
     Everything process-shaped lives in the central protocol — do not copy it here. -->

# <!-- TODO: Project Name -->

<!-- TODO: 2-3 sentences — what this is, the stack, how it's deployed. -->

profile: standard   <!-- prototype | standard | production -->
verifier: none      <!-- e.g. codex-mcp, "@codex review" PR comment, none -->
flow: feature -> main   <!-- branch topology; e.g. feature -> staging -> main.
                             Branch from (and PR into) the FIRST integration
                             branch. Promotion PRs (staging -> main) are gated
                             on constituent-PR greenness, not re-verified. -->

## Process

This project follows the **lu-product-os** protocol: outcome gates, tiers, PR-based
delivery, independent verification on full-tier changes.
Read `lu-product-os-EXECUTION-PROTOCOL.md` in the lu-product-os repo before building.
Run `lu-product-os-doctor --quick` at session start and state the machinery banner.

## Commands

```bash
./scripts/lu-product-os-verify   # full quality gates (G1) — CI runs exactly this
# TODO: dev server, migrations, and other project commands
```

## Key Rules

<!-- TODO: only project-specific constraints that apply to ALL work. Keep short. -->

- Surface blockers immediately — never work around a missing tool/credential silently.

## Docs

<!-- TODO: pointers only. For workspace projects: canonical docs live in the
     workspace repo — do not create or maintain docs in this repo. -->

## Progress

<!-- TODO: phase checkboxes, or "direct development only". -->
