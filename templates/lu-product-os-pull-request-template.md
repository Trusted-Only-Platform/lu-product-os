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

**You are the independent verifier.** Everything you need is in this PR — work from the
criteria above and the diff. Do not explore the repo beyond the diff and the files it touches.

**First, read your depth from the pr-check summary on this PR: `G3 depth: full` or `G3 depth: review-only`.**

**If review-only** (infra/dependency escalation — Dockerfiles, workflows, dependency bumps):
read the acceptance criteria, review the diff hunks (correctness, security implications of the
infra or dependency change, compliance with the project's rules docs), and post your verdict.
Write no tests. Run nothing — CI has already proven the suite green.

**If full:**

1. **Without reading the existing test files first**, write acceptance tests covering each
   criterion above, plus the edge and error cases that matter. Commit them to this branch
   as `test:` commits.
2. Run **only the tests you wrote**. Do not re-run the full suite — CI has already proven it
   green (gate G1), and CI will re-run everything including your tests after you push.
3. Review the diff hunks: correctness, unnecessary abstraction, dead code, compliance with
   the project's rules docs.
4. **Constraints:** modify only test files; report source problems as PR comments, do not
   fix them yourself.
5. Post your verdict as a PR comment, first line exactly one of:
   `VERDICT: approve` or `VERDICT: request-changes` — followed by your findings.

**Re-reviews:** after a request-changes round, read only the commits pushed since your last
verdict and check them against your findings — do not re-review the whole PR.
