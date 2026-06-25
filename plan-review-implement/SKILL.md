---
name: plan-review-implement
description: "Full-cycle engineering workflow: plan → review → implement → verify. Use when the user asks to implement a significant feature, refactor, or change. Activates a structured process of exploration, planning, independent review, surgical implementation, and verification."
---

Follow this structured workflow for any non-trivial implementation task. The process ensures shared understanding, prevents regressions, and delivers verified code.

## Phase 1 — Discovery & Alignment

1. **Explore the codebase.** Use the Explore subagent, file_search, grep_search, and read_file to understand relevant existing code, analogous features that can serve as implementation templates, and potential blockers or ambiguities. For tasks spanning multiple areas (frontend + backend, different repos), launch 2–3 Explore subagents in parallel.

2. **Grill the user.** Use the `grilling` skill to interview the user relentlessly about every aspect of the plan until a shared understanding is reached. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide a recommended answer. Ask one question at a time — never multiple questions simultaneously. If a question can be answered by exploring the codebase, explore the codebase instead.

3. **Do NOT write code yet.** If doubts arise during discovery, stop and ask. Surface assumptions, ambiguities, and tradeoffs before committing to a design.

## Phase 2 — Write the Plan

4. **Draft the implementation plan** in `/memories/session/plan.md` (use the `memory` tool). The plan must include:
   - A TL;DR (what, why, recommended approach)
   - Step-by-step implementation with explicit dependencies — mark which steps can run in parallel and which block on prior steps
   - For plans with 5+ steps, group into named phases that are each independently verifiable
   - Exact file paths to create or modify, referencing specific functions/types/patterns
   - Explicit scope boundaries — what's included and what's deliberately excluded
   - Verification steps (specific tests, commands, manual checks — not generic statements)
   - Decisions: confirmed assumptions, tradeoffs, and approach rationale

5. **Test-first regression guard.** When touching/altering/refactoring existing code used by already-working functionality:
   - If tests are missing for the code being changed, write characterization tests FIRST that pin the current behavior, get them green on unchanged code, then change and keep them green
   - Prefer EXTENSION over modification: don't mutate existing code when you can add new code alongside it
   - For new meaningful code, include a minimal set of backend and frontend tests

6. **If doubts arise while writing the plan, stop and ask the user.** Do not make large assumptions.

## Phase 3 — Skeptical Independent Plan Review

7. **Spin up at least one independent subagent** to skeptically review the plan against requirements and existing code. The subagent(s) should be given the full plan and told to verify assumptions against real code. For complex tasks, spawn multiple subagents in parallel:
   - One for frontend plan review
   - One for backend plan review
   - Optionally, a third for strategy/architecture review when the logic is new, abstract, or complex

   Each subagent must report: (A) verdict (sound / sound-with-caveats / has-blocking-issues), (B) blocking issues with file:line evidence, (C) non-blocking caveats/risks, (D) confirmations verified, and (E) concrete suggestions. They are SKEPTICAL reviewers — their job is to find problems before implementation.

8. **Fix/update the plan** based on subagent(s) feedback. Re-run any subagent review if the plan changes significantly.

## Phase 4 — Present & Get Approval

9. **Present the final plan to the user.** Show the plan directly (not just a file path). The plan stays in `/memories/session/plan.md` for persistence.

10. **Wait for explicit approval** before writing any code. Do NOT start implementation until the user says "go", "implement", "OK", or equivalent.

## Phase 5 — Implement

11. **Implement following the plan.** Use the `surgical-developer` skill for guidance: think before coding, prefer simplicity, make surgical (minimal) changes, surface assumptions, and define verifiable success criteria. Important: re-use existing code patterns and conventions. Avoid speculative abstractions or unrequested features. For example, if an existing frontend component or pattern already does what you are implementing—or just needs a minor refactor—use that instead of writing it from scratch.

12. **Keep changes additive where possible.** If shared code must change, apply the test-first regression guard from Phase 2.

13. **Add tests** for new meaningful code (backend integration/unit tests, frontend component tests). Follow existing test conventions (naming, location, harness).

## Phase 6 — Verify & Self-Review

14. **Run all verification steps** before declaring completion:
    - Backend: `cd backend && mvn test` — full suite must be green
    - Frontend: `cd frontend && npm test` — all tests must pass
    - Frontend build: `cd frontend && npm run build` — must succeed (typecheck + lint + build)
    - All verification steps from the plan must pass

15. **Review the implementation.** Use the `skeptical-review` skill to spin up skeptical subagents that review the finished CODE against requirements, codebase conventions, and the plan. At least one subagent; multiple for complex tasks (frontend reviewer + backend reviewer).

16. **Fix any issues** the subagent(s) find. Re-run tests/build after fixes.

## Phase 7 — Report

17. **Report back to the user** with:
    - A summary of what was implemented
    - The main files created and modified
    - Test results (backend, frontend)
    - Any caveats, known limitations, or deferred items
    - Reference to the plan document for full details

## Key Principles

- **Extension over modification.** Don't mutate shared code when you can add alongside it.
- **No silent assumptions.** If something is unclear, stop and ask.
- **Test-first on shared-code changes.** Pin existing behavior before altering it.
- **Independent review at two checkpoints.** Plan review (before code) and code review (after implementation).
- **User approval gate.** Never write code before the plan is approved.
- **Surgical changes.** Minimal code, no speculative abstractions, no unrequested features.
