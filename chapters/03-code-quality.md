---
layout: page
title: "Code Quality — Every Word Is Spelled Correctly"
---

# Code Quality — Every Word Is Spelled Correctly

*Your existing proofreading approaches won't help you when every word is spelled correctly.*

Here's the problem: AI-generated code is fluent. It follows conventions, uses reasonable variable names, and passes syntax checks. Traditional code review — the kind where you scan for typos, missing semicolons, obvious logic errors — catches almost nothing. Every word is spelled correctly. The bugs are in the logic, the architecture, the subtle assumptions.

This is the same problem that appeared when voice dictation replaced typing. Every word was a real word, in the dictionary, spelled perfectly. The proofreading tools built for typos were useless. You needed tools that understood *meaning*, not just *spelling*.

AI-generated code needs the same shift. You need tools that understand structure and intent, not just syntax and style.

---

## Stage 1: Lint (day one)

This is the absolute minimum. Set it up before you write any code, and run it in CI so it's enforced.

A linter catches the mechanical problems: unused variables, unreachable code, inconsistent formatting, imports from wrong paths. These aren't the interesting bugs, but they're the noise that makes the interesting bugs harder to find.

- [ ] Linter configured and running in CI
- [ ] Linter runs before every commit (pre-commit hook or review skill)
- [ ] Linter failures block merge
- [ ] Rules tuned to your project — disable the ones that generate false positives, keep the ones that catch real issues

**What this catches:** The 80% of problems that are mechanical. Formatting inconsistencies, unused imports, basic type errors.

**What this misses:** Everything interesting. A linter won't tell you that your authentication middleware is missing, that you're querying a column that doesn't exist, or that the same business logic is duplicated in three files.

## Stage 2: Static analysis in CI (week two)

Add a static analysis tool to your CI pipeline. CodeQL, Semgrep, or equivalent. These tools understand code flow, not just syntax — they can trace data from user input to database query and flag injection vulnerabilities.

- [ ] Static analysis tool running on every PR
- [ ] Security-focused rules enabled (injection, XSS, SSRF, path traversal)
- [ ] Findings triaged — not every alert is real, but every alert gets a decision

**What this catches:** Security vulnerabilities that look syntactically correct. A SQL query that concatenates user input. An API endpoint that redirects to a user-controlled URL. Dead code that's technically reachable.

**What this misses:** Architectural problems. It can tell you this line is vulnerable, but not that your entire auth pattern is inconsistent across 30 routes.

**Note:** Static analysis tools like CodeQL are a bridge. They provide value early when you have no project-specific checks. Once you have AI review skills (stage 3) that understand your project's context and conventions, generic static analysis catches less and less that the review skills don't already flag. At that point, evaluate whether it's still earning its CI minutes or just generating noise you've learned to ignore.

## Sidebar: AI review tools — buy vs build

Before building your own review skills, look at the ecosystem of AI code review tools. Many offer free tiers that are genuinely useful:

- **CodeRabbit** — AI code review on every PR. Understands context, suggests improvements, catches logic issues. Free for open source, paid for private repos.
- **CodeAnt** — automated code review with security focus. Free tier available.
- **Aikido** — security scanning (SAST, secrets, dependencies) with GitHub integration. Free tier covers core scanning.

These tools give you AI-powered review on day one with zero setup beyond installing a GitHub app. They're especially valuable early, when you don't yet have project-specific conventions to encode into custom skills.

The progression:
1. **Start with a tool** — install CodeRabbit or equivalent, get AI review immediately
2. **Add custom review skills** — as your project develops conventions the tool doesn't know about, encode them into your own review skills that run alongside the tool
3. **Evaluate the overlap** — over time, your custom skills may cover enough that the paid tool tier isn't adding proportional value. Or the tool may keep catching things your skills miss. Either is fine.

Remember the tool economics principle from chapter 1: many of these tools charge per PR or per seat. Review locally before creating PRs to reduce how many PRs hit the paid scanner. This isn't gaming the system — local review catches the easy stuff so the paid tool focuses on what it's uniquely good at.

## Stage 3: AI review skills (when your codebase has conventions worth enforcing)

Once your project has established patterns — how auth works, where files go, what the PR format looks like — encode them into review skills that run on every commit and every PR.

This is where you address the "every word is spelled correctly" problem directly. The review skill understands your project's conventions and checks whether new code follows them, not just whether it compiles.

- [ ] Review skill runs before every commit
- [ ] Review checks project conventions (not just generic code quality)
- [ ] Review checks security patterns (auth wrappers, input validation)
- [ ] Review evolves as you discover new failure patterns — when something gets through, add a check

**The test accumulation problem:**

Your AI agent will generate tests. Lots of them. Most of these tests, by default, will be "does the thing run without crashing" — they verify that the code executes, not that it does the right thing. These tests pass, they pad your coverage numbers, and they catch almost nothing.

Meaningful tests — the kind that actually prevent regressions — require thinking about what the code *should* do, not just what it *does*. Test-driven development forces this thinking, but it takes discipline and a deliberate workflow. Don't confuse test volume with test value. Ten tests that verify behavior boundaries are worth more than a hundred tests that verify the code doesn't throw.

- [ ] Tests verify behavior, not just execution
- [ ] Coverage numbers are understood for what they actually measure (and don't measure)
- [ ] When a bug gets to production, a regression test is written that would have caught it

## Stage 4: Structural analysis (when your codebase is complex enough to have invisible problems)

At some point your codebase crosses a threshold where no single person — human or AI — can hold the whole structure in their head. Individual files look fine. The architecture is drifting. Problems are invisible when you review file by file.

This is where structural analysis tools come in. Think of it as an MRI for your codebase: it parses the entire codebase into a dependency graph, applies annotation codes to flag anomalies, and surfaces problems that are invisible at the file level.

What a codebase MRI finds:
- **Missing auth middleware** — 30 API routes with no auth wrapper. Are they intentionally public, or bugs?
- **Duplicated logic** — the same pattern implemented slightly differently in 9 files
- **Dead code** — files imported by nothing, exports used by no one. Unreviewed attack surface.
- **God files** — 2,000-line files with 40+ methods that nobody can review thoroughly
- **Architecture drift** — code structure contradicts what the architecture docs describe
- **Coupling violations** — imports crossing domain boundaries
- **Scattered configuration** — the same value hardcoded in 8 files instead of centralized

None of these are visible when reviewing a single PR. They only appear when you analyze the codebase as a structure.

- [ ] Structural analysis tool built or adopted
- [ ] Runs periodically (nightly or weekly) with results diffed against previous scan
- [ ] New anomalies generate work items routed to the appropriate owner
- [ ] Integrated into PR review — changed files checked against the structural graph

**When structural problems meet security:**

Tech debt is security exposure. Dead code is unreviewed, unmonitored attack surface. God files are too large to audit thoroughly. Missing auth on routes is guaranteed to be found — if not by your security tools, then by someone else's. The correct assumption in an AI era is that the probability of detection is 1 for everything.

Cross-referencing structural findings with threat models (attack trees or equivalent) tells you which tech debt items are security-critical and which are merely annoying. An unused file on no attack path is low priority. A missing auth wrapper on a route that leads to token theft is critical.

## Stage 5: Autonomous code quality agent (when you have enough rules and patterns)

The final stage: a dedicated agent whose only job is code quality. It runs against its own set of rules, it picks up findings from structural analysis, and it creates PRs to fix mechanical issues — dead code removal, lint fixes, consistency cleanup.

The constraint: it never changes behavior. If fixing a finding requires understanding what the code should do, it routes a work item to the appropriate owner instead of fixing it itself.

- [ ] Dedicated code quality agent with its own worktree/workspace
- [ ] Works from a rules catalog — each rule has clear scope and examples
- [ ] One PR per rule type (not one mega-cleanup PR)
- [ ] Never changes behavior — mechanical fixes only, routes everything else
- [ ] Lowest priority — never blocks feature work

---

## The progression

| Stage | When | What it catches |
|-------|------|----------------|
| Lint | Day one | Mechanical — formatting, unused vars, basic errors |
| Static analysis | Week two | Security — injection, XSS, data flow vulnerabilities |
| AI review skills | When you have conventions | Convention violations, project-specific patterns |
| Structural analysis | When the codebase is too big to hold in your head | Architecture drift, coupling, dead code, ownership gaps |
| Autonomous agent | When you have enough rules to automate | Mechanical cleanup at scale, routed findings |

Don't skip stages. Each one builds on the last. And don't mistake volume of checks for quality of checks — a linter that generates 200 warnings nobody reads is worse than no linter at all.

---

## Signals you've outgrown this chapter

- Your structural analysis is finding cross-team issues that need coordination
- Your code quality agent is generating more work items than it's fixing itself
- You need to prioritize tech debt against feature work systematically

When this happens, you're ready for the throughput and team coordination chapters.
