---
layout: post
title: "Security in the Mythos Era"
permalink: /security-mythos/
---

# Security in the Mythos Era

*Probability of detection = 1. Everything will be found. Plan accordingly.*

## The paradigm shift

For decades, security operated on the assumption that most vulnerabilities would never be found. A bug in a parsing library that had been there for 11 years? Probably safe. An obscure code path that nobody had looked at? Not a priority.

That assumption is dead. AI-assisted vulnerability discovery is finding decades-old bugs at unprecedented rates. Researchers and attackers alike are using AI to scan codebases, chain together subtle vulnerabilities, and discover exploit paths that were invisible to manual review. A package with no CVE history isn't safe — it's unexamined.

The correct assumption for everything in your stack: **probability of detection = 1.** If a vulnerability exists, it will be found. The only question is whether you find it first.

## What this changes

### Dead code is attack surface

In the old model, dead code was tech debt — annoying, cleanup it when you get to it. In the Mythos era, dead code is unreviewed, unmonitored attack surface. Nobody is watching it for new vulnerability patterns. Nobody is testing it. But it's deployed and reachable.

Remove it. If it's not in your codebase, it can't be exploited.

### God files are unauditable

A 2,000-line file with 40+ methods is too large for any human or AI to audit thoroughly in a single pass. Vulnerabilities hide in complexity. The structural analysis tools from the code quality chapter (codebase MRI) identify these — break them up.

### Silence from a package is a warning sign

This is the most counterintuitive shift. Traditionally, no CVEs = good. In the Mythos era, no CVEs on a package that processes untrusted input = suspicious.

A package that parses PDFs, processes HTML, or handles binary formats should be generating CVE disclosures proportional to its complexity and age. If it's not, either:
- It's receiving active security scrutiny and is genuinely well-maintained (check: does it have an OpenSSF Scorecard? Active security policy? Recent patches?)
- Nobody is looking at it, and its "Mythos moment" — a sudden burst of critical disclosures — is coming

### Tech debt is security exposure

Every item from the code quality chapter that you deferred? In the Mythos era, it's a security item:

| Tech debt | Security exposure |
|-----------|------------------|
| Dead code | Unmonitored attack surface |
| God files | Too complex to audit |
| Missing auth wrappers | Guaranteed to be found |
| Coupling violations | Paths that cross security boundaries |
| Stale code (unchanged while siblings evolved) | Nobody watching for new CVE patterns |
| Duplicated logic | Fix in one place, vulnerability persists in the copy |

---

## The Attentiveness Quotient: predicting where risk hides

Traditional dependency scanning (Dependabot, `pnpm audit`, Aikido) is reactive — it alerts you when a CVE is published. That's necessary but insufficient. You also need to predict which packages are likely to have their Mythos moment.

The Attentiveness Quotient (AQ) is a framework for estimating whether a package is receiving security attention proportional to its risk surface.

### The core metric: Velocity Ratio

```
Expected annual CVEs = base_rate x code_size x security_surface x age_factor x AI_multiplier
Velocity Ratio (VR) = Observed CVEs (12 months) / Expected CVEs (12 months)
```

| VR Range | Interpretation |
|----------|----------------|
| 0.6 - 1.5 | Healthy — proportional disclosure activity |
| 0.3 - 0.6 | Monitor — possibly under-scanned |
| < 0.3 | Suspicious silence — investigate |
| > 2.0 | Actively targeted or has hygiene issues |

The security surface multiplier matters most:
- **High surface** (1.0x): auth, crypto, HTTP, payments, parsers handling untrusted input
- **Medium surface** (0.4x): data utilities, state management
- **Low surface** (0.1x): build tools, UI components, dev-only

A PDF parser with zero CVEs in 6 years has a VR of 0.00. That's not reassuring — that's alarming.

### What to do about quiet packages

For each dependency with VR < 0.3 and high security surface:

```
Is it processing untrusted input?
├── No → Accept + Monitor
└── Yes
    ├── Is a more-scrutinized substitute available?
    │   ├── Yes, worth the migration cost → Substitute
    │   └── No → Audit + Monitor
    └── Can you sandbox it? (isolated function, no credentials)
        ├── Yes → Sandbox + Audit
        └── No → Audit NOW
```

Real examples of this decision:
- **PDF parser with VR 0.00** → substitute with a library maintained by a browser security team (Mozilla's pdfjs-dist vs. a thin wrapper around an old fork)
- **DOCX parser with VR 0.25** → no good substitute; already runs in an ephemeral serverless function (sandboxed); queue a security audit
- **HTML-to-text converter with VR 0.00** → accept + monitor; output is plaintext and can't carry exploits
- **NLP parser with VR 0.00** → accept + monitor; only processes already-extracted text, not raw input

### The composite score

The full Supply Chain Attentiveness score combines three dimensions:

| Dimension | Weight | What it measures |
|-----------|--------|-----------------|
| Velocity Ratio | 40% | CVE disclosure rate vs. expected |
| Time-to-Patch | 35% | How fast disclosed CVEs get fixed (target: Critical <7 days, High <14 days) |
| Process Score | 25% | OpenSSF Scorecard signals — branch protection, code review, dependency pinning |

---

## The Mythos Wave playbook

When a previously quiet package suddenly gets its first AI-discovered CVE cluster — and this will happen to packages in your dependency tree — here's what to do:

**Immediate (day zero):**
- Patch to latest, even if it's a major version bump
- Check if the disclosed CVEs affect your specific usage patterns

**48 hours:**
- Expect 2-5 more CVEs as other researchers pile on (AI-discovered clusters come in waves)
- Budget time for rapid patching

**1 week:**
- Reassess: is the maintainer responding? Is the patch cadence sustainable?

**1 month:**
- If patching cadence is good and maintainer is responsive: stay
- If maintainer is overwhelmed or unresponsive: plan migration to a substitute

---

## The arms race

Attackers use AI to find vulnerability chains. Defenders use AI to find them first. As a solo founder or small team, you can't out-resource an attacker. But you can:

1. **Minimize surface area** — less code, fewer dependencies, smaller attack surface
2. **Predict where risk concentrates** — AQ identifies the quiet packages before they explode
3. **Automate detection** — layered scanning tools (SAST, SCA, review skills) running continuously
4. **Make the easy targets hard** — auth on every route, secrets out of code, RLS on every table. Attackers optimize for easy wins; making the easy paths hard forces them to find harder ones

You don't need to be unhackable. You need to not be the easiest target.

---

## Operational integration

### Quarterly AQ audit

Every quarter, run the AQ assessment:

- [ ] Extract metadata for each direct dependency: code size, age, observed CVEs
- [ ] Compute Velocity Ratio for each
- [ ] Flag anomalies: any high-surface package with VR < 0.3 gets a manual review
- [ ] Check for Mythos moments: did any previously quiet package suddenly get 3+ CVEs?
- [ ] Update risk tiers and decisions

### Continuous signals

Between quarterly audits, watch for:

- [ ] Dependabot/Aikido alerts (reactive baseline)
- [ ] New CVE on a VR < 0.3 package (treat as high priority — the package just "woke up")
- [ ] Maintainer change on a critical dependency (supply chain takeover signal)
- [ ] Unexpected version publish (check for compromised releases)

---

## Signals you're ready for more

- Your quarterly audits are routine and you want to formalize the ongoing practice
- You need to triage security findings when you're one person with limited time
- You want to build a sustainable security practice, not just respond to incidents

When this happens, you're ready for the security operations chapter.
