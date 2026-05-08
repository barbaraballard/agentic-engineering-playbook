# Agentic Engineering Playbook

From vibe-coded prototype to professional product.

## What this is

A collection of patterns for building production software with AI coding agents. Each chapter captures hard-won lessons from shipping a real product this way — the kind of knowledge that's easy to develop and easy to forget.

## Who it's for

**Primary audience:** Future-me (Barbara), starting a new product. Hand these chapters to Claude (or your AI coding agent of choice) at project start and skip the rediscovery phase.

**Secondary audience:** Builders who have a working prototype — built with Bolt, Lovable, Cursor, or pure vibe coding — and want to turn it into something professional. You don't need "how to prompt an AI." You need "how to run a software operation where AI does most of the coding."

## How to use it

**With an AI coding agent:** Add the relevant chapters to your project context. They're written to work as bootstrap documents — enough context that your agent knows the patterns, pitfalls, and workflows without you re-explaining everything.

**As a human reader:** Browse the chapters in order if you're starting fresh, or jump to the stage chapter that matches where you are.

## Chapters

### Foundation (read in order)

| # | Chapter | What it covers |
|---|---------|---------------|
| 1 | [Getting Started](chapters/01-getting-started.md) | Ground-clearing decisions and the five foundations contract |
| 2 | [Code Quality](chapters/02-code-quality.md) | Why traditional review fails for AI-generated code, and what to do instead |

### Stages (pick up when ready)

These chapters are independent. Reach for them when the need arises.

| Chapter | When you need it |
|---------|-----------------|
| Token Economics | Your AI costs are rising or your context window feels cramped |
| API Tokens & Secrets | You have your first API key and nowhere safe to put it |
| Multiplying Throughput | Your single-agent workflow is a bottleneck |
| Running a Team of Agents | You need specialized roles, not just more hands |
| Application Security | You have users and something worth protecting |
| Support Operations | You're the entire support team |
| Finance Operations | You have revenue and need to track costs |

*Stage chapters are added as patterns are captured. Not all are written yet.*

## Principles

- **Principles first, tools second.** Tools change. The patterns underneath them don't.
- **Written for agents, readable by humans.** Every chapter works as Claude bootstrap context *and* as a reference you can browse.
- **Living documents.** Updated when new patterns emerge. The "what good looks like" sections evolve as we learn.
- **YAGNI.** Each chapter covers the minimum you need at that stage, not everything you might eventually want.

## Customizing for your project

The `our-context/` directory is `.gitignored` by default. Put your organization-specific or project-specific variants there: notes, checklists, tweaks to the guidance. Anything in `our-context/` that you don't distribute is entirely yours.

## License & how to customize this

The core library (all files in this repository except `our-context/`) is licensed under the [Mozilla Public License 2.0 (MPL-2.0)](LICENSE).

In practice, that means:

- You are free to use this library in your own projects, including commercial ones.
- If you modify the library files themselves and then redistribute those modified versions (for example, by publishing a fork on GitHub or shipping them as part of a product), you must make those modified files available under MPL-2.0 as well.
- You can keep your own separate files and project-specific content private or under a different license.

The `our-context/` directory is `.gitignored` by default. Put your organization-specific or project-specific variants there. Anything you keep in `our-context/` and do not distribute is entirely yours; the MPL obligations only apply when you distribute modified versions of the licensed files in this repo.

If you're unsure whether your planned use counts as "distribution" under MPL-2.0, you should talk to a lawyer or your legal team.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Improvements to the shared library are welcome.