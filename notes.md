# Playbook Notes

Capture file for "oh yeah and then there is this" moments. Sorted roughly by which chapter they belong to. Move to chapters when writing.

---

## Running a Team of Agents

- Worktrees aren't just parallel capacity — they have **specific areas of responsibility**. Security-claude owns security alerts. Ops-claude owns monitoring. When a Dependabot storm hits, security-claude is already working on it because that's its job. You didn't triage or assign — you already dispatched by defining the roles.

- **Migration ID per worktree** eliminates migration conflicts. With multiple agents creating migrations simultaneously, filename collisions were constant and tedious to decouple. Solution: give every worktree a 2-digit migration ID baked into the filename (`YYYYMMDDWWSSSS`). Now conflicts only happen when there's an actual bug, not a naming collision. You don't need 6 digits of migrations per day.

- **PR titles include the worktree name.** When something breaks, you know who to ask — the worktree that authored the PR is right there in the title. `[security] fix auth bypass` or `dashboard: add school cards`. Simple convention, huge payoff for traceability.

- **Move heavy review to the worktree, not the PR.** The original workflow: agents write code, create PRs, PRs go through heavy automated review (Claude, CodeRabbit, Aikido, etc.), findings require fixes, fixes require re-review, merge conflicts pile up, triggering yet another review cycle. Result: "code-writing days" and "PR days" as separate activities, massive GitHub Actions minutes burn, and forced upgrades to paid tiers on every review tool. The fix was inverting where the work happens:
  1. All heavy review (`/review` skill with 8 parallel passes) runs **locally in the worktree** before creating a PR — where multiple agents can work in parallel without conflicts
  2. Only one agent works on the actual PR phase at a time (lightweight serialization)
  3. GitHub's required checks are fast: QA test suite + a **review receipt** (a file generated during `/review` that CI checks for). The receipt isn't security against malicious actors — it's a gate that prevents eager agents from skipping the review they were supposed to run
  4. Result: PRs are small, pre-reviewed, fast to merge. Review tools run locally (free tier), not on GitHub (paid tier). Merge conflicts dropped dramatically.

## (Uncategorized)

