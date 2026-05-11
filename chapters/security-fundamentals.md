# Security Fundamentals

*Security isn't a feature you add later. It's a way of thinking about where things can go wrong.*

You don't need to be a security expert. You need to understand where risk lives, how to detect it, and what tools exist so you're not starting from zero when something goes wrong.

---

## The four layers where risk lives

Every security problem in your application lives in one of four layers. When something feels wrong, ask "which layer?" first.

### Layer 1: Code

Your code — and the code your AI agent writes. This is where most people start and stop thinking about security.

- **Auth bypass** — routes without authentication, broken session handling
- **Injection** — SQL injection, XSS, SSRF, path traversal
- **PII exposure** — personal data in logs, error messages, API responses
- **Encryption gaps** — sensitive data stored in plaintext

### Layer 2: Supply Chain

Every package you import is code you didn't write and don't review. Your `package.json` is an attack surface.

- **Known vulnerabilities** — published CVEs in your dependency tree
- **Maintainer compromise** — a trusted package gets taken over (this happens — axios had a state-actor compromise in March 2026)
- **Transitive risk** — a vulnerability three levels deep in your dependency tree that you've never heard of
- **Abandoned packages** — no maintainer, no patches, still in your build

### Layer 3: Configuration

How your infrastructure is configured. The code can be perfect and the config can still be wrong.

- **Database permissions** — overly broad grants, missing row-level security
- **Environment variables** — production secrets in staging, default passwords
- **Network exposure** — services accessible that shouldn't be
- **RLS policies** — always-true policies, missing policies on sensitive tables

### Layer 4: Vendor Health

You depend on services. When they go down or get compromised, you're affected.

- **Service outages** — your auth provider, database host, or payment processor goes down
- **Vendor security incidents** — your provider gets breached
- **API changes** — breaking changes that affect your security assumptions
- **Certificate/credential expiry** — things that stop working silently

When you audit your security posture, walk all four layers. Most people check layer 1 and stop. Layers 2-4 is where the surprises live.

---

## Attack trees: thinking like an attacker

An attack tree is the single most useful security thinking tool. It answers: "what are all the ways someone could achieve [bad thing]?"

Start with a goal an attacker would have:
- Steal user credentials
- Access another user's data
- Compromise your infrastructure
- Abuse your payment system

Then work backward: what paths lead to that goal? Each path has steps. Each step has controls (things that block it) or is open (nothing stops it).

```
Goal: Access another family's data
├── Via broken RLS policy
│   ├── Control: RLS policies on all user-data tables (PRESENT)
│   └── Control: RLS audit in /audit-security (PRESENT)
├── Via auth bypass on API route
│   ├── Control: withFamilyAuth middleware (PRESENT on most routes)
│   └── OPEN: 3 routes identified without auth wrapper
└── Via LLM prompt injection
    ├── Control: de-identification before LLM (PRESENT)
    └── Control: output validation (PARTIAL)
```

The value isn't the diagram — it's the thinking. You discover the 3 unprotected routes not by reading code, but by systematically asking "how could this goal be achieved?"

- [ ] Identify 3-5 attacker goals relevant to your product
- [ ] Map the paths to each goal
- [ ] Mark each step as OPEN, PRESENT (control exists), or VERIFIED (control tested)
- [ ] Prioritize: OPEN paths on high-value goals first

Store your attack trees in version control. Update them when you add new surfaces (new API routes, new integrations, new data types). They're living documents, not a one-time exercise.

**What good looks like later:**
- Attack tree assessment integrated into your PR review process — every PR checked against relevant trees
- New P0 paths (open + high value) are ship-blockers
- Cross-referencing with your codebase structural analysis to find unprotected routes automatically

---

## The acronyms that matter

Security tooling has a lot of jargon. Here's what actually matters for a product developer:

| Acronym | What it does | When you need it | Example tools |
|---------|-------------|-----------------|---------------|
| **SAST** | Static Application Security Testing — scans your source code for vulnerabilities | Week one. Run on every PR. | Aikido, Semgrep, CodeQL |
| **SCA** | Software Composition Analysis — scans your dependencies for known CVEs | Week one. Automated alerts. | Dependabot, `pnpm audit`, Aikido |
| **DAST** | Dynamic Application Security Testing — tests your running app for vulnerabilities | When you have a staging environment to test against | OWASP ZAP, Burp Suite |
| **SBOM** | Software Bill of Materials — a list of everything in your build | When compliance requires it, or when you need to answer "are we affected by CVE-X?" fast | `pnpm sbom`, Syft |

**Start with SAST + SCA.** These are free or cheap, automated, and catch the most common issues. DAST is valuable but requires a running environment and more setup. SBOM is a compliance/incident-response tool — useful when you need it, not a daily driver.

The buy-vs-build decision from the code quality chapter applies here too:

1. **Start with vendor tools** — Aikido, Dependabot, CodeRabbit all have free tiers and catch real issues immediately
2. **Add project-specific checks** — your review skills can check for patterns the generic tools don't know about (your auth middleware convention, your encryption patterns)
3. **Layer, don't replace** — each tool catches different things. Pre-commit hooks catch secrets. SAST catches injection. SCA catches vulnerable deps. Attack tree reviews catch design-level gaps. No single tool covers everything.

---

## Layered review: your actual defense

The way security works in practice is layers of review, each catching different classes of problems. One project runs eight parallel review passes on every PR:

| Pass | What it checks | Tool type |
|------|---------------|-----------|
| CodeRabbit AI | General code quality + logic | Vendor AI review |
| Code Quality & Logic | Bugs, race conditions, error handling | Custom review skill |
| Security | Auth bypass, injection, PII exposure, input validation | Custom review skill |
| Project Conventions | Auth middleware usage, logging patterns, API response format | Custom review skill |
| Silent Failures | Empty catch blocks, unchecked errors, fire-and-forget calls | Custom review skill |
| Postgres Best Practices | Query performance, RLS, migration safety | Specialized skill |
| Aikido SAST & Secrets | Static analysis + hardcoded secrets | Vendor SAST |
| Attack Tree Impact | Does this PR open/close/weaken an attack path? | Custom review skill |

You don't need eight passes on day one. You might start with just a linter and Aikido. But the principle is: **each layer catches what the others miss.** A SAST tool doesn't know your auth convention. Your convention checker doesn't know about SQL injection patterns. The attack tree pass doesn't care about code style — it cares about whether you just opened a new path to credential theft.

Build up layers as you discover what gets through your existing checks.

- [ ] At least one SAST tool running on PRs (Aikido, CodeQL, or equivalent)
- [ ] At least one SCA tool running (Dependabot, pnpm audit)
- [ ] Pre-commit secret scanning (bettersecrets)
- [ ] Project-specific security checks in your review skill (auth patterns, PII handling)
- [ ] Attack trees exist for your top 3-5 attacker goals

**What good looks like later:**
- Eight or more parallel review passes on every PR
- Attack tree impact assessment integrated into review
- New security patterns added to review skills when incidents reveal gaps
- Different models reviewing each other's work (different blind spots)

---

## Signals you're ready for more

- You have attack trees and they're current, but you're wondering about your dependencies
- A package you depend on just got a cluster of CVEs and you were caught off guard
- You're asking "are we actually safe, or just lucky?"

When this happens, you're ready for the Mythos chapter: probability of detection = 1.
