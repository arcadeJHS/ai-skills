---
name: skeptical-review
description: "Skeptical independent code review via subagents. Use when the user asks for a code review, wants implementation validated against requirements/plan/conventions, or needs a quality gate before merging. Works on any codebase — no prior plan document required."
---

Spin up independent, skeptical subagents to review finished code against requirements, codebase conventions, and (if available) an implementation plan. Each subagent is a SKEPTIC — its job is to find problems, not to praise the code.

## When to Use

- User asks for a code review ("review this", "check my code", "audit")
- User wants implementation validated before merging
- User wants a quality gate on an existing codebase
- As Phase 6 of the `plan-review-implement` workflow (post-implementation review)

## Step 1 — Gather Context

Before launching reviewers, collect the essential context:

1. **If a plan document exists** (e.g., `/memories/session/plan.md`), read it. The plan is the benchmark for correctness.
2. **If no plan exists**, ask the user for 2–3 sentences describing what the code is supposed to do. This becomes the de facto requirements spec.
3. **Identify the codebase conventions**: file_search for existing patterns (test file naming, component structure, error handling style, etc.) that reviewers should check against.
4. **Determine scope**: what files/folders should be reviewed? If the user doesn't specify, use the `get_changed_files` tool to identify changed files, or fall back to running `git diff --name-only` in the terminal.

## Step 2 — Spin Up Review Subagents

Launch at least one skeptical reviewer subagent. For complex tasks, launch multiple in parallel:

| Reviewer | Focus | When to Use |
|----------|-------|-------------|
| **Code reviewer** | Correctness, edge cases, error handling, security, code quality | Always |
| **Frontend reviewer** | Component structure, styling consistency, accessibility, state management | When frontend code exists |
| **Backend reviewer** | API contracts, data integrity, performance, auth/authz | When backend code exists |
| **Strategy/Architecture reviewer** | Design patterns, abstractions, coupling, scalability | For new/complex/abstract logic |

Each subagent must receive:
- The full plan or requirements description
- The file paths to review
- The codebase conventions to check against
- The instruction: **"You are a SKEPTICAL reviewer. Your job is to find problems, not praise. Be thorough and specific — cite file:line evidence for every issue."**

## Step 3 — Required Subagent Report Format

Every reviewer subagent must return a structured report with exactly these sections:

**(A) Verdict** — one of:
- `sound` — no issues found
- `sound-with-caveats` — non-blocking concerns only
- `has-blocking-issues` — must fix before proceeding

**(B) Blocking Issues** — each with `file:line` evidence. Issues that would cause bugs, regressions, security vulnerabilities, or violate core requirements. If none, state "None."

**(C) Non-Blocking Caveats/Risks** — performance concerns, maintainability nits, missing edge-case handling, style inconsistencies. If none, state "None."

**(D) Confirmations Verified** — specific claims checked against real code (e.g., "Confirmed the API endpoint returns 404 for missing resources — test at `api.test.ts:45` covers this").

**(E) Concrete Suggestions** — actionable fixes for each blocking issue and the most important caveats. Be specific: what to change, where, and why.

## Step 4 — Synthesize & Report

After all subagents return, synthesize their findings into a single review summary:

1. **Overall verdict**: blocking-issues trumps caveats trumps sound
2. **Consolidated blocking issues list** (deduplicated across reviewers)
3. **Consolidated caveats list**
4. **Recommended fix order** (what to address first)

Present this synthesis directly to the user. If there are blocking issues, ask whether to fix them now or defer.

## Step 5 — Optional: Fix Issues

If the user wants fixes applied, follow these principles:
- Fix blocking issues first, then high-impact caveats
- Keep changes surgical — touch only what's needed
- Re-run the relevant subagent(s) after fixes to verify resolution

## Key Principles

- **Skeptical, not complimentary.** Reviewers hunt for problems. "Looks good" is not a finding.
- **Evidence-based.** Every issue must cite `file:line`.
- **Independent verification.** Reviewers check claims against real code, not assumptions.
- **Actionable output.** Every issue comes with a concrete fix suggestion.
- **Works standalone.** No dependency on `plan-review-implement` or any prior workflow.
