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

## Security / Guardrails

- **Security patches and feature work need different pipelines.** Features flow dev-first (beta → main). Security patches flow prod-first (main → beta). Same repo, same branch protection, opposite direction. The trigger: a Next.js update fixed 22 CVEs (8 high), merged to beta at 3 AM, but production wouldn't get the fix until the next manual promotion — potentially weeks. Solution: dependabot and security-only patches target `main` directly, auto-merge forward to `beta` via a PR (not a direct push — the PR provides a canary if the patch breaks in-flight feature work). Feature PRs continue targeting `beta` with full review. Three review tiers:
  - **Tier 1 (dep bumps):** automated, zero human review — build + test + Aikido scan. Patch/minor auto-merge; major requires manual review.
  - **Tier 2 (security code fixes touching only deps/config/CI):** local security review pass + build/test, lighter than full beta gate.
  - **Beta (unchanged):** full 8-pass review receipt for all feature work.
  - The routing rule is simple: if the fix only touches deps, config, or CI → main-first. If it touches application logic → beta with full review.

- **Accepted risk tracking.** Some vulnerabilities can't be fixed (transitive dep locked by another package). Track them in a structured file (`accepted-risks.json`) with: package, version, reason, accepted date, and review date. Quick security checks skip accepted risks; quarterly audits surface them for re-evaluation. The review date triggers a reminder so accepted risks don't become forgotten risks.

## Cross-Cutting / Playbook Intro

- **We keep reinventing established engineering principles with an AI-first twist.** Least privilege, defense in depth, separation of duties, patch management, threat modeling, fail-safe defaults — these all exist in textbooks. But the implementation is completely different when your "developer" is an LLM that pushes 80 actions per hour and routes around any obstacle that isn't structural. This isn't limited to security — it's happening across code quality (proofreading → structural analysis), operations (monitoring → attention-based alerting), team coordination (standup meetings → shared memory + announcements), and workflow (waterfall/agile → parallel agents with review gates). The patterns come from a combination of human skills (product design, systems thinking, domain expertise, human factors awareness) and LLM skills (speed, breadth, tirelessness, code generation). Neither alone arrives at these solutions. The human brings the "what worked before in a different context" and the LLM brings the "here's how to implement it at machine scale." The playbook is the record of what emerges from that collaboration.

## Positioning / Voice

- **The playbook is terse by design.** It gives you enough to do it yourself. But it also makes clear how deep the rabbit hole goes. Every chapter is a surface-level treatment of something that's been built and operated at scale. The reader gets principles and checklists. If they want someone who's lived it — that's a conversation.

- **Human factors and AI factors.** The playbook sits at the intersection of two disciplines: understanding how humans actually behave (habituation, attention decay, signal-vs-noise, workaround-seeking) and understanding how AI agents actually behave (confident mistakes at machine speed, routing around obstacles, hallucinating schema, optimizing for completion over safety). Neither discipline alone produces these solutions. The agentic engineering practice emerges from designing systems that account for both.

## (Uncategorized)

