---
layout: home
title: Home
---

# Agentic Engineering Playbook

From vibe-coded prototype to professional product.

A collection of patterns for building production software with AI coding agents. Each chapter captures hard-won lessons from shipping a real product this way — the kind of knowledge that's easy to develop and easy to forget.

## Who it's for

**Primary audience:** Future-me, starting a new product. Hand these chapters to Claude (or your AI coding agent of choice) at project start and skip the rediscovery phase.

**Secondary audience:** Builders who have a working prototype — built with Bolt, Lovable, Cursor, or pure vibe coding — and want to turn it into something professional. You don't need "how to prompt an AI." You need "how to run a software operation where AI does most of the coding."

## How to use it

**With an AI coding agent:** Add the relevant chapters to your project context. They're written to work as bootstrap documents — enough context that your agent knows the patterns, pitfalls, and workflows without you re-explaining everything.

**As a human reader:** Read the foundation chapters in order if you're starting fresh, or jump to the stage chapter that matches where you are.

## Foundation chapters (read in order)

These build on each other. Start here.

| # | Chapter | What it covers |
|---|---------|---------------|
| 1 | [Getting Started](chapters/01-getting-started) | Ground-clearing decisions and the six foundations contract |
| 2 | [Guardrails](chapters/02-guardrails) | Prevent disasters from humans and LLMs — environment pipeline, database controls, secret hygiene |
| 3 | [Code Quality](chapters/03-code-quality) | Why traditional review fails for AI-generated code, and what to do instead |

## Stage chapters (pick up when ready)

These are independent. Reach for them when the need arises.

| Chapter | When you need it |
|---------|-----------------|
| Token Economics | Your AI costs are rising or your context window feels cramped |
| API Tokens & Secrets | You have your first API key and nowhere safe to put it |
| Multiplying Throughput | Your single-agent workflow is a bottleneck |
| Running a Team of Agents | You need specialized roles, not just more hands |
| [Security Fundamentals](chapters/security-fundamentals) | You have users and something worth protecting — layers, attack trees, tooling |
| [Security in the Mythos Era](chapters/security-mythos) | You depend on open-source packages and probability of detection = 1 |
| Security Operations | You need ongoing security practice, not just initial setup |
| Support Operations | You're the entire support team |
| Finance Operations | You have revenue and need to track costs |

*Stage chapters are added as patterns are captured. Not all are written yet.*

## Principles

- **Principles first, tools second.** Tools change. The patterns underneath them don't.
- **Written for agents, readable by humans.** Every chapter works as Claude bootstrap context *and* as a reference you can browse.
- **Living documents.** Updated when new patterns emerge. The "what good looks like" sections evolve as we learn.
- **YAGNI.** Each chapter covers the minimum you need at that stage, not everything you might eventually want.
