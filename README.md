# adversarial-reviewer

An Agent Skill that forces genuinely critical code reviews by adopting three adversarial personas. Breaks the self-review monoculture where AI reviewing its own code produces "LGTM" on everything.

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

## Why This Exists

Standard AI code review produces three failure modes:
- **Self-review blindness** — same model, same blind spots
- **Politeness bias** — hedging instead of direct findings
- **Shallow coverage** — checking syntax but missing logic, security, and maintainability

This skill addresses all three by requiring adversarial personas that cannot produce "no issues found."

## License

MIT
