# adversarial-reviewer

An Agent Skill that forces genuinely critical code reviews by adopting three adversarial personas. Breaks the self-review monoculture where AI reviewing its own code produces "LGTM" on everything.

## Latest: Security Advisory Filed on Axios

**[GHSA-8wrj-g34g-4865](https://github.com/axios/axios/security/advisories/GHSA-8wrj-g34g-4865)** — Filed a GitHub Security Advisory on axios (108K stars, 65M weekly downloads) after the maintainer requested formal disclosure. The README's `beforeRedirect` example bypasses `follow-redirects`' credential-stripping on protocol downgrades (CWE-319). A [fix PR](https://github.com/axios/axios/pull/10624) is now open.

**[Presented to the Node.js Security Working Group](https://github.com/nodejs/security-wg/issues/1560)** — Filed as an ecosystem-level pattern: 4 npm libraries (180M+ weekly downloads) with secure code defaults that teach insecure patterns in their documentation.

**[Your Dependencies Have Two Attack Surfaces](https://gist.github.com/ekreloff/ef8b442fa856eb3fa275b367d56cbb7e)** — Comparative analysis of supply chain vs. documentation attack surfaces in the npm ecosystem.

## The Problem

When Claude (or any AI) reviews code it just wrote, it shares the same mental model and blind spots as the author. Users consistently report this as a top frustration:

> "Claude reviewing Claude's code has the same blind spots as Claude writing the code."

This skill forces a perspective shift through three hostile reviewers — The Saboteur (production failures), The New Hire (readability), and The Security Auditor (vulnerabilities) — each required to find at least one real issue.

## Install

### Via skills CLI

```bash
npx skills add ekreloff/adversarial-reviewer
```

### Via git clone

```bash
git clone https://github.com/ekreloff/adversarial-reviewer.git ~/.claude/skills/adversarial-reviewer
```

### Via curl (one-liner)

```bash
mkdir -p ~/.claude/skills/adversarial-reviewer && curl -sL \
  https://raw.githubusercontent.com/ekreloff/adversarial-reviewer/main/SKILL.md \
  -o ~/.claude/skills/adversarial-reviewer/SKILL.md
```

### Manual

Copy `SKILL.md` into your project's `.claude/skills/` directory or your global `~/.claude/skills/` directory.

## Usage

```
/adversarial-review              # Review staged/unstaged changes
/adversarial-review --diff HEAD~3  # Review last 3 commits
/adversarial-review --file src/auth.ts  # Review a specific file
```

## What It Does

1. **Gathers changes** from git diff or specified files
2. **Reads full context** — not just changed lines, but entire files
3. **Runs three adversarial personas**, each MUST find at least one issue:
   - **The Saboteur** — "How do I break this in production?"
   - **The New Hire** — "Can I understand this with zero context?"
   - **The Security Auditor** — OWASP-informed vulnerability scan
4. **Deduplicates and promotes** — issues caught by 2+ personas get promoted in severity
5. **Produces structured output** with BLOCK/CONCERNS/CLEAN verdict

## Example Output

```markdown
## Adversarial Review: Add user authentication middleware

**Scope:** src/middleware/auth.ts (47 lines added), src/routes/api.ts (3 lines changed)
**Verdict:** CONCERNS

### Warnings
1. **[Saboteur]** `verifyToken()` catches all exceptions and returns `null` — a network
   timeout during token verification silently grants unauthenticated access (line 23)
2. **[Security Auditor → promoted from NOTE]** JWT secret loaded from `process.env.JWT_SECRET`
   with fallback to hardcoded `"development"` — if env var is missing in production,
   all tokens are signed with a known key (line 8)

### Notes
1. **[New Hire]** `validateRequest()` also modifies `req.user` — the name suggests
   validation-only but it has a side effect that callers must know about

### Summary
The auth middleware has a silent failure mode that could grant unauthenticated access
under network issues. The JWT secret fallback is a deployment risk. Fix the exception
handling in `verifyToken()` before merge.
```

## Real-World Results

This tool has been used to review production code in four high-profile npm libraries totaling ~180 million weekly downloads:

| Library | Downloads/week | Finding | Issue |
|---------|---------------|---------|-------|
| [axios](https://github.com/axios/axios) | 65M | beforeRedirect example bypasses credential-stripping (CWE-319) | [GHSA](https://github.com/axios/axios/security/advisories/GHSA-8wrj-g34g-4865), [#10614](https://github.com/axios/axios/issues/10614), [Fix PR](https://github.com/axios/axios/pull/10624) |
| [node-jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) | 76M | Regex audience matching without anchor enforcement | [#1019](https://github.com/auth0/node-jsonwebtoken/issues/1019) |
| [cors](https://github.com/expressjs/cors) | 25M | README regex example allows CORS origin bypass | [#408](https://github.com/expressjs/cors/issues/408) |
| [multer](https://github.com/expressjs/multer) | 13.5M | README example uses Math.random() instead of crypto | [#1386](https://github.com/expressjs/multer/issues/1386) |

These reviews revealed a systemic pattern: **libraries with secure code defaults that teach insecure patterns in their documentation.** Read the full analysis: [The Documentation Attack Surface](https://gist.github.com/ekreloff/2c44e97183a74c32fdbb7d14aa8b30ad).

## Request a Review

Want your code reviewed by three adversarial personas?

[**Open a review request**](https://github.com/ekreloff/adversarial-reviewer/issues/new?template=review-request.yml)

| Tier | Price | Includes |
|------|-------|----------|
| Open-source | Free | Findings published as public gist |
| Single file | $5 | Private delivery of focused review |
| Full library | $25 | Private delivery + written security report |

For paid reviews, payment details are provided after scoping.

## Why This Exists

Standard AI code review produces three failure modes:
- **Self-review blindness** — same model, same blind spots
- **Politeness bias** — hedging instead of direct findings
- **Shallow coverage** — checking syntax but missing logic, security, and maintainability

This skill addresses all three by requiring adversarial personas that cannot produce "no issues found."

## License

MIT
