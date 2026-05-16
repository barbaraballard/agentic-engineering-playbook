---
layout: post
title: "Getting Started — The Foundations Contract"
permalink: /getting-started/
order: 1
description: "Ground-clearing decisions and the six foundations contract — source control, project instructions, security basics, CI, hosting, and a PR pipeline from day one."
---


*Before you add another feature, you will have these foundations in place.*

You have a working prototype — built with Bolt, Lovable, Cursor, or pure vibing. It does something real. Now you want to make it a product.

This chapter is the contract. Do these things before writing another line of feature code.

One thing to internalize: you are building two things at once. Your product, and the development system that builds your product. Both will evolve. Your CI pipeline, your review process, your project instructions, your agent workflows — these aren't setup tasks you do once. They're a second product that grows alongside the first. The foundations in this chapter are version 1 of your development system. Later chapters are the upgrades.

---

## Part 1: Ground-Clearing Decisions

### Assess portability

If you built on a platform (Bolt, Lovable, etc.), decide now: stay or migrate.

Migration is painful even when the stack looks identical. The code is shaped by the platform's opinions — file structure, deployment model, state management. The longer you wait, the more expensive the move.

The test: does the platform support branch-based workflows, code review, CI/CD, environment separation, and parallel workstreams? That last one matters more than you'd think — prototyping tools envision a single thread with a backlog. Once you're running multiple agents (or even just want to work on two things at once), you need a tool that supports it. Git worktrees, for example, let you run independent workstreams against the same repo without branch-switching. If your platform can't do this, you'll hit the ceiling fast.

Migrate now while the codebase is small.

- [ ] Evaluated whether current platform supports branch workflows, CI/CD, and parallel workstreams
- [ ] Made a stay-or-migrate decision
- [ ] If migrating: completed migration before adding new features

### Don't use npm

If you're in the JavaScript/TypeScript ecosystem, switch to pnpm (or Bun) now. npm has structural problems that get worse as your project grows:

- **Phantom dependencies** — npm's flat hoisting lets you import packages you never declared. Your code works until a transitive dep changes and it silently breaks. pnpm's strict isolation catches this immediately.
- **Supply chain exposure** — npm has been hit by repeated large-scale supply chain attacks. pnpm's content-addressable store and strict lockfile integrity make tampering harder to hide.
- **Disk and speed with multiple workstreams** — if you're running parallel agents in worktrees, npm installs a full `node_modules` per worktree. pnpm's shared store means the second worktree installs in seconds, not minutes.

This is a small migration early. It's a painful migration later.

- [ ] Using pnpm (or Bun) instead of npm
- [ ] Lockfile committed and CI uses `--frozen-lockfile`

### Choose your hosting model

Serverless is often right. But check these before committing:

- **Cold starts** — acceptable for your use case?
- **Execution time limits** — do you have background jobs or long-running tasks?
- **Statelessness** — do you need WebSockets or persistent connections?
- **Vendor lock-in** — how platform-specific are the APIs you're using?
- **Cost curve** — cheap now, but what happens at 10x traffic?

A container or a VPS is also fine. The point is to choose deliberately.

This decision extends beyond hosting into your entire platform ecosystem. Run the cost-per-user math early — AI model costs, database, hosting, and auth add up fast on some stacks. One founder was set up on Google Cloud (Gemini + Firebase + GCP hosting) and facing $4/user/month before writing any business logic. Switching to a cheaper LLM, an open-source database, and a simpler hosting platform cut costs by 90%. That kind of savings changes what's viable as a business. Do this analysis before you've built too much to move. This is the viability test that kills most LLM wrapper products — they pick the most capable model, the most convenient platform, and discover too late that the unit economics don't work.

- [ ] Evaluated hosting tradeoffs for your specific workload
- [ ] Ran cost-per-user math across the full stack (AI, database, hosting, auth)
- [ ] Made a deliberate platform choice (not just the default or the familiar one)

### Set up backups

A cron job that dumps your database somewhere. A script that zips and uploads. Whatever. Janky is fine.

- [ ] Backup process exists (any form)
- [ ] Tested restoring from backup at least once

### Map your tool costs

The free tier of most tools (GitHub, Supabase, Netlify, Sentry) is enough to start and lasts longer than you'd expect.

Know where the paid walls are. Many tools charge at the PR layer — CI minutes, code review, deploy previews. You can stay in free tiers longer by catching issues locally before creating PRs.

This isn't about avoiding paying. It's about not being surprised.

- [ ] Identified which tools you're using and their free-tier limits
- [ ] Know where the first paid threshold is for each tool

---

## Part 2: The Six Foundations

### 1. Source control with a review gate

*Nothing ships unreviewed.*

**Do today:**
- [ ] Code lives in a Git repository (GitHub, GitLab, etc.)
- [ ] Branch protection enabled on `main` — no direct pushes allowed
- [ ] Every change goes through a pull request, even small ones
- [ ] Every PR gets at least one review before merge — an AI review skill counts, and is a good starting point

**What good looks like later:**
- Multiple reviewers — AI review catches patterns, human spot-checks intent
- Review skills that check security, quality, and project conventions automatically

### 2. Project instructions

*Your AI agent should know what you know about this project.*

Your prototype probably has no project instructions, or minimal ones. Every conversation starts from scratch. Fix this first — it's the single highest-leverage file in your repo.

**Do today** — create a CLAUDE.md (or equivalent) containing:
- [ ] What the app does, what stack it uses, and where the main pieces live
- [ ] How the pieces connect — what calls what, where data flows
- [ ] Naming conventions and patterns to follow
- [ ] What not to do — past mistakes, known pitfalls, patterns the agent gets wrong

**What good looks like later:**
- Living document that evolves with the project
- References to specialized instruction files for specific areas
- Captures "why" decisions, not just "what"
- A new team member (human or AI) can get productive by reading it

### 3. Security basics

*Secrets in your git history are there forever. Set this up before your first commit.*

AI agents are fast and careless with secrets. They'll paste an API key into a config file, commit it, and move on. You need guardrails before that first commit happens, not after.

**Do today:**
- [ ] Secrets in environment variables, not in code — `.env.local` (or equivalent) for local dev, hosting platform's env var management for production. Note: `.env.local` is plaintext on disk. It's fine for getting started, but consider starting with a secrets manager instead (1Password CLI with `op run`, Doppler, or Infisical all have free tiers). Migrating away from `.env.local` gets harder the more secrets and environments you accumulate — easier to start right than to switch later
- [ ] `.gitignore` includes `.env*`, credentials files, and anything that shouldn't be committed
- [ ] Pre-commit or pre-push secret scanning hook — [bettersecrets](https://github.com/nicholasgasior/bettersecrets) catches leaked keys before they reach the repo
- [ ] Auth from a provider (Supabase Auth, Auth0, Clerk, etc.) — don't build your own authentication; DIY auth is where security goes to die
- [ ] HTTPS enabled — usually free from your hosting platform, just confirm it's on

**What good looks like later:**
- Layered scanning: pre-push hook + CI scanner + GitHub app (e.g., Aikido)
- Dependency vulnerability scanning in CI (`pnpm audit` or equivalent)
- Attack trees as a design tool for thinking about threats
- A full security chapter in this playbook

### 4. CI pipeline

*The pipeline is the quality floor.*

Start small. A linter and a build check are enough on day one. The point is that an automated gate exists and is enforced — you'll add to it over time.

**Do today:**
- [ ] CI configuration that triggers on every pull request
- [ ] Linting — catches style issues and common errors before you look at the code
- [ ] Build verification — confirms the code at least compiles and bundles
- [ ] Merge blocked when CI fails — this is the enforced part; a CI that's advisory only will get ignored

**What good looks like later:**
- Test suite running in CI
- Security scanning (dependency vulnerabilities, secret detection)
- Fast enough that it doesn't slow you down

### 5. Hosting with environments

*If you can't deploy it, you can't ship it.*

You need a hosting environment, not just a deploy target. The difference is observability and control.

**Do today — a hosting environment with:**
- [ ] Automatic deploys from `main` — merge triggers deploy, no manual steps
- [ ] Deploy logs — when something breaks after a deploy, you need to see what went out and when, without guessing
- [ ] Environment variables — secrets and configuration managed by the hosting platform, not hardcoded or committed
- [ ] A real URL — your app is accessible to users, not running on localhost

**What good looks like later:**
- Production + preview/staging environments
- Preview deploys on PRs so you can see changes before they go live
- Environment variables scoped per environment (staging secrets != production secrets)
- Rollback capability — even if it's just "revert the commit and auto-redeploy"

### 6. PR pipeline from day one

*The habits you set now are the habits you'll have at scale.*

This is where the other five come together into a single workflow:

1. Work on a branch
2. Review before committing
3. Create a PR
4. CI runs
5. Review the PR
6. Merge
7. Auto-deploy

**Do today:**
- [ ] A review step runs before every commit — a review skill, a linter, something that checks your work before it becomes permanent
- [ ] PRs for every change, no exceptions — even "just a quick fix" goes through the pipeline
- [ ] CI runs on every PR — automated checks are the baseline, not a nice-to-have
- [ ] You actually use this process every time — a pipeline you skip "just this once" is a pipeline you don't have

**What good looks like later:**
- Review skills checking project conventions, security, and code quality
- AI-generated PR descriptions that explain what changed and why
- PR template that prompts for a test plan
- Git history that tells the story of how the project evolved

---

## Signals you're ready for more

- [ ] You're context-switching between unrelated tasks and wish you could delegate
- [ ] Your single agent conversation is the bottleneck
- [ ] You want to work on a feature while something else is in review
- [ ] Your instructions file is getting long because there's too much to know

When this happens, you're ready for the throughput chapter: parallel agents, worktrees, subagents.

Don't rush it. A single well-instrumented agent with good foundations will outperform a team of agents with no discipline.
