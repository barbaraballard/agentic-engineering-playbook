# Playbook Notes

Capture file for "oh yeah and then there is this" moments. Sorted roughly by which chapter they belong to. Move to chapters when writing.

---

## Running a Team of Agents

- Worktrees aren't just parallel capacity — they have **specific areas of responsibility**. Security-claude owns security alerts. Ops-claude owns monitoring. When a Dependabot storm hits, security-claude is already working on it because that's its job. You didn't triage or assign — you already dispatched by defining the roles.

- **Migration ID per worktree** eliminates migration conflicts. With multiple agents creating migrations simultaneously, filename collisions were constant and tedious to decouple. Solution: give every worktree a 2-digit migration ID baked into the filename (`YYYYMMDDWWSSSS`). Now conflicts only happen when there's an actual bug, not a naming collision. You don't need 6 digits of migrations per day.

- **PR titles include the worktree name.** When something breaks, you know who to ask — the worktree that authored the PR is right there in the title. `[security] fix auth bypass` or `dashboard: add school cards`. Simple convention, huge payoff for traceability.

## (Uncategorized)

