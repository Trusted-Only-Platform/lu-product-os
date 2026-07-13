<!-- lu-product-os PR template. The PR is the deliverable and the permanent record.
     pr-check (CI) fails this PR if Acceptance Criteria are missing, if UI changes
     lack screenshots, or if a full-tier change has no verifier verdict. -->

## What & Why

<!-- 2-4 sentences: what this change does and the problem it solves. -->

Ticket: <!-- e.g. fixes TRU-42, or "direct change — scope stated below" -->

## Acceptance Criteria

<!-- Copy from the plan (phase file / feature plan / ticket). Given/When/Then.
     These are the definition of done — the verifier tests against them. -->

- Given … when … then …

## Screenshots

<!-- Required if the diff touches UI paths. Drag images here. Delete section for backend-only changes. -->

## Deviations from plan

<!-- Anything built differently than planned, and why. "None" is a fine answer. -->

---

## Verification Brief <!-- for the independent verifier (full tier) -->

**You are the independent verifier.** Everything you need is in this PR.

1. **Without reading the existing test files first**, write acceptance tests covering each criterion above, plus edge and error cases. Commit them to this branch as `test:` commits.
2. Run the full suite: `./scripts/lu-product-os-verify`
3. Review the diff: correctness, unnecessary abstraction, dead code, compliance with the project's rules docs.
4. **Constraints:** modify only test files; report source problems as PR comments, do not fix them yourself.
5. Post your verdict as a PR comment, first line exactly one of:
   `VERDICT: approve` or `VERDICT: request-changes` — followed by your findings.
