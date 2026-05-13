---
layout: post
title: "Guardrails — Preventing Disasters"
permalink: /guardrails/
order: 3
description: "Eleven plays for preventing humans and LLMs from causing irreversible harm — environment pipelines, database controls, secret hygiene, and the habituation problem."
---

# Guardrails — Preventing Disasters

*Humans make mistakes. LLMs make mistakes. Build processes and systems so that neither can cause a disaster.*

There are numerous stories of AI agents deleting production databases. Not because the AI was malicious — because it was asked to "clean up" and nobody had built a fence around the thing that mattered most. The LLM doesn't know what's irreversible. You have to encode that knowledge into your systems.

There's a deeper problem: the human in the loop isn't as reliable as you think. Humans are bad at sustained attention — it's a well-studied human factors problem. We habituate to stimulus-response patterns. The LLM asks "should I proceed?", you hit return. It does this eighty times an hour and on the eighty-first time the answer should have been no, but your finger was already moving. Signal gets lost in noise. Vigilance decays.

This is why guardrails must be structural, not behavioral. "Be careful" is not a guardrail. A system that won't let the agent write to the production database — that's a guardrail.

A concrete example: your AI coding tool asks permission before running database queries. The permission prompt is a 7-line block of text. The difference between a SELECT and a DROP TABLE is on line 2 or 3. At eighty confirmations an hour, you are not reading line 2. You are hitting return. The fix isn't "read more carefully" — it's a pre-tool hook that blocks DDL keywords before the prompt ever reaches you. The dangerous action becomes impossible, and you can safely approve the routine ones without vigilance fatigue.

This extends to your agent's entire permission model. Tune it in two directions simultaneously: **allow more safe actions** (file reads, searches, linting — reduce the noise) and **hard block dangerous actions** (DDL on production, force push, branch deletion — remove the prompt entirely so there's nothing to accidentally approve). Less noise, better signal, and the catastrophic actions aren't gated by your attention — they're gated by a wall.

This chapter is a set of plays. Each one addresses a specific category of mistake that will happen if you don't prevent it.

---

## Play 1: Review before every commit

*The cheapest place to catch a mistake is before it becomes permanent.*

Your AI agent will generate more code in a day than you can read. You need a review step that runs before every commit — not after, not at the PR stage, but before the code enters your git history at all.

A review skill (like `/review`) that checks for security issues, project convention violations, and obvious errors is your first line of defense. It's fast, it's automated, and it catches the things that would otherwise show up as "wait, when did this break?"

- [ ] Review skill or equivalent runs before every commit
- [ ] Review checks for security issues, not just style
- [ ] Review is non-negotiable — no "just this once" skips

**What good looks like later:**
- Reviews from independent agents with different perspectives — a security agent catches things a frontend agent doesn't, and vice versa. One project runs nine specialized agents; the review power comes from independence, not repetition
- Reviews from different models. An agent built on Claude reviewing code written by GPT (or vice versa) catches different classes of mistakes than self-review. Different models have different blind spots — use that to your advantage
- Review skills that evolve as you discover new failure patterns
- Review before commit AND review on the PR — two different checkpoints catching different things

## Play 2: Environment pipeline

*Never test in production. Build a pipeline with gates between environments.*

You need separation between where you develop, where you test, and where your users are. The minimum is two environments. The ideal is four:

```
local → staging → beta → prod
```

Each environment has its own database (or database branch). The critical principle: **staging has its own data that you can destroy without consequence.** Your AI agent can experiment, run migrations, and break things in staging. Production is untouchable.

Human gates at key transitions:
- **staging → beta**: human reviews the PR, confirms it works in staging
- **beta → prod**: human confirms beta has been stable, merges to main

Your hosting platform and database provider likely support this. Supabase has database branching. Netlify and Vercel have preview deploys. Use them.

The key: make every promotion a manually triggered action, not an automatic one. One project's pipeline has four manual gates:
- **apply-migrations-to-staging** — manually triggered; schema changes hit staging first
- **promote-to-beta** — manually triggered; moves code from staging to beta environment
- **publish-beta** — manually triggered; the build already exists, this just makes it live
- **publish-main** — manually triggered; promotes to production

Nothing moves toward users without a human deciding it should. The LLM can build, test, and prepare — but the "go live" button is always yours.

One more trick: set your development branch (not production) as the default branch in GitHub. Every PR, every `gh pr create`, every AI-generated pull request will target the safe environment by default. Pushing to production requires a deliberate choice to change the target. This single setting prevents an entire class of "oops, that went straight to prod" accidents.

- [ ] At least two environments (staging + production)
- [ ] Each environment has its own database or database branch
- [ ] Staging data is disposable — you can nuke it without fear
- [ ] Human approval required to promote changes toward production
- [ ] AI agents develop against staging, never directly against production

**What good looks like later:**
- Four environments: local, staging, beta, production
- Preview deploys for every PR
- Database branches that mirror production schema but contain test data
- Automated smoke tests that run after each promotion

## Play 3: Database access controls

*The LLM should not have write access to your production database. Period.*

This is the play that prevents the "AI deleted the production database" story from being your story.

Your AI coding agent needs to read production data sometimes — to debug, to understand schema, to verify a fix. It should never be able to write to it, and especially never be able to delete from it.

Build a read-only database MCP (or equivalent tool) for your local AI agent. It can query production to get the data it needs. It cannot INSERT, UPDATE, DELETE, or DROP anything in production. For writes, it uses the staging database — which you can rebuild from scratch if something goes wrong.

- [ ] Local AI agent has no write access to production database
- [ ] Read-only MCP or tool for production data access
- [ ] AI agent writes only to staging database
- [ ] Staging database can be rebuilt from scratch (and you've tested this)
- [ ] `execute_sql` and similar tools are restricted to read-only queries on production
- [ ] No production database credentials in your local `.env.local` — if the connection string isn't there, the agent can't use it. Local development uses staging credentials only.

**What good looks like later:**
- Separate database credentials per environment, scoped to minimum necessary permissions
- Audit logging on all production database access
- AI agent can't even accidentally target production — the connection string isn't available locally

## Play 4: Migrations through the pipeline, never direct

*Never apply a database migration directly. Every schema change goes through a PR.*

Your AI agent will happily run `execute_sql` with a `CREATE TABLE` or `ALTER COLUMN` directly against your production database if you let it. This bypasses version control, skips review, and means your migration history doesn't match your actual schema.

Every DDL change (CREATE, ALTER, DROP) must be a migration file in git, go through a PR, pass review, and deploy through your CI/CD pipeline. Use `execute_sql` for read-only queries and one-off data fixes only.

- [ ] All schema changes are migration files in version control
- [ ] Migrations go through PRs with review
- [ ] `execute_sql` restricted to SELECT queries on production
- [ ] AI agent instructed in project instructions to never apply migrations directly

**What good looks like later:**
- Migration naming convention that prevents collisions across workstreams
- CI that validates migrations before they can merge
- Rollback migrations for every forward migration

## Play 5: Secrets never in conversation

*Conversations are an attack surface. Secrets should never appear in them.*

Your AI agent will cheerfully display API keys, tokens, and passwords in its conversation output if you don't tell it not to. This is a problem because conversations may be logged, cached, or visible in ways you don't control.

Instruct your agent in your project instructions: read secrets silently, suppress output. Never echo a token value, never include a key in a status update, never display credentials "for verification."

Then add defense in depth: pre-commit secret scanning (bettersecrets) catches any that leak into code, and pre-push hooks catch anything that slipped through.

- [ ] Project instructions explicitly say "never display secret values in conversation"
- [ ] Pre-commit or pre-push secret scanning hook installed
- [ ] AI agent reads secrets from environment variables, never from hardcoded values
- [ ] Tested: asked the agent to show you an API key and confirmed it refused

**What good looks like later:**
- Layered scanning: pre-commit hook, CI scanner, GitHub app
- Secret rotation procedures documented
- Secrets manager instead of `.env.local` files

## Play 6: AI hallucinates schema (and other facts)

*The AI will invent database columns, API endpoints, and configuration options that don't exist. It will do this with complete confidence.*

This is one of the most subtle failure modes. Your AI agent will write a query referencing a column that sounds right, looks right, and doesn't exist. The code will look correct in review. It will fail at runtime.

The defense: before writing SQL or modifying schema, verify against the actual database. Don't trust the agent's memory of the schema — it's a hallucination waiting to happen.

How schema verification works in practice: give your agent a read-only tool that can query the database's information schema. Before writing a query that references `user_schools.enrollment_date`, the agent checks whether `enrollment_date` actually exists on `user_schools`. This can be a database MCP with a `list_tables` / `describe_table` tool, a generated TypeScript types file that stays in sync with your schema, or even just a project instruction that says "run `SELECT column_name FROM information_schema.columns WHERE table_name = '...'` before writing any query against a table you haven't verified this session."

- [ ] Project instructions: "verify column/table existence before writing SQL"
- [ ] Database MCP or tool available for schema verification (list tables, describe columns)
- [ ] Consider generating and committing a schema types file that the agent can reference without a database round-trip
- [ ] Tests that exercise database queries (catches hallucinated columns at test time, not in production)

## Play 7: Never bypass safety hooks

*The LLM's first instinct when a pre-commit hook fails is `--no-verify`. Never allow this.*

When a hook catches a problem — a failing test, a leaked secret, a lint error — the fix is to fix the problem, not to skip the hook. Your AI agent doesn't understand this instinct. It sees a blocked commit as an obstacle and reaches for the flag that removes the obstacle.

Your project instructions should explicitly prohibit `--no-verify`, `--no-gpg-sign`, `--force`, and similar bypass flags unless you specifically request them. The hooks are there because you put them there. They're not suggestions.

- [ ] Project instructions prohibit `--no-verify` and similar bypass flags
- [ ] Project instructions prohibit `git push --force` to main/production branches
- [ ] If a hook fails, the agent fixes the underlying issue rather than skipping the check

## Play 8: Destructive action confirmation

*Irreversible operations require a human in the loop.*

Some operations can't be undone: force pushing, dropping tables, deleting branches, deleting user data, resetting to a previous state. Your AI agent should never perform these autonomously.

Build a confirmation gate — a skill, a prompt, a workflow step — that pauses before any destructive action and requires explicit human approval. The agent describes what it's about to do, you confirm, then it proceeds.

- [ ] Destructive operations require explicit human confirmation
- [ ] Agent describes the action and its consequences before asking for approval
- [ ] Production branch (`main`) is explicitly excluded from deletion, force-push, and reset operations
- [ ] Project instructions list which operations are considered destructive

**What good looks like later:**
- A `/destructive-action` skill that enforces the confirmation pattern
- Audit log of destructive actions taken and who approved them

## Play 9: LLMs will find the workaround and make it the default

*The LLM optimizes for task completion. Every gate you build, it will try to route around.*

This is the most insidious failure mode because it looks like productivity. The LLM isn't being malicious — it's doing what you asked (finish the task) by removing what's in the way (your safety checks). Watch for these patterns:

**"Test failures not related to our code"** — the LLM runs tests, some fail, and it declares them irrelevant and moves on. Maybe they are irrelevant. Maybe they're catching a real regression your change introduced. If there's no system to capture and track test failures, the LLM will train you to ignore them.

**"It's just two lines of code"** — the change is small, so the LLM skips review and jumps straight to creating a PR (or worse, straight to applying). Small changes break things too. Add structural friction: a review receipt that CI checks for, so a PR physically cannot merge without evidence that a review ran.

**"--SKIP"** — the LLM discovers that your pipeline has escape hatches and starts using them by default. `SKIP_PRE_PUSH=1`, `--no-verify`, `[skip ci]` in commit messages. Once it works once, the LLM will reach for it every time.

The defense is structural, not instructional. You can tell the LLM "don't skip tests" in your project instructions, and it will comply until the moment skipping is the fastest path to completion. Instead:

- [ ] Gates are enforced in CI, not just in convention — if review didn't run, the PR can't merge
- [ ] Test failures are captured and tracked, not just logged — a failing test requires a decision, not a dismissal
- [ ] Escape hatches (`--no-verify`, `SKIP_*` flags, `[skip ci]`) are documented in project instructions as prohibited
- [ ] When the LLM proposes skipping something, treat it as a signal that the thing it wants to skip might be catching a real problem

## Play 10: Don't auto-apply AI-generated fixes

*AI security scanners, GitHub autofix PRs, and dependency bots all generate "fixes." Review every one.*

Your toolchain will increasingly generate automated fix suggestions — GitHub AI autofix PRs, security scanner remediation, dependency updates. These need different levels of scrutiny:

**Auto-merge is fine for:** Dependabot version bumps and similar dependency updates, as long as CI passes. These are mechanical changes — if the build succeeds and tests pass, the update is safe. Auto-merging these keeps your dependencies current without burning review cycles.

**Always review:** AI-generated code fixes (GitHub autofix PRs, security scanner remediation, code quality suggestions). These change behavior, not just versions. The fix can be worse than the finding: a security "fix" that changes control flow, an autofix that patches the symptom and hides the root cause, a refactoring suggestion that breaks an edge case.

- [ ] Dependency version bumps auto-merge when CI passes
- [ ] AI-generated code fixes are always reviewed before merging
- [ ] Security remediation PRs are reviewed by someone who understands the vulnerability

---

## Play 11: Tell the agent when something is a regression

*If it used to work, say so. Otherwise the agent will debug from scratch and may rewrite stable code.*

When something breaks, your instinct is "X is broken, fix it." The agent takes that at face value and treats it as a greenfield debugging problem — it reads the code, forms a theory, and starts changing things. If the code is complex, it may decide the "fix" is a rewrite of something that was working fine yesterday.

One word changes everything: **regression.** "X is a regression — it was working before" tells the agent to focus on what changed recently, not on re-understanding the entire system. It will check recent commits, look at recent diffs, and find the change that broke it. This is almost always faster and less destructive than a from-scratch investigation.

- [ ] When reporting bugs to your agent, specify whether it's new behavior or a regression
- [ ] If it's a regression, tell the agent approximately when it last worked (or which PR/commit you suspect)
- [ ] Project instructions: "When told something is a regression, check recent changes first before investigating the broader codebase"

---

## Signals you've outgrown these plays

- You have multiple agents working in parallel and need coordination guardrails (not just individual ones)
- Your environment pipeline needs automated promotion (not just human gates)
- You need formal incident response procedures for when guardrails fail

When this happens, you're ready for the operations and security stage chapters.

The plays in this chapter don't go away — they're the foundation. Everything else builds on top of "the AI can't delete production."
