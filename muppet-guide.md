# Muppet Guide — Example

This file is an iterative training guide for a muppet. The muppet reads it on every startup and follows these accumulated instructions. Over time, this becomes an encoding of your judgment — like onboarding a team lead who gets better with every performance review.

**How to use:** Copy this file into your project directory (or keep a per-project version). After each muppet run, review what went well and what didn't, then add entries below. The muppet reads this alongside its prompt and adjusts its behavior accordingly.

---

## Decomposition Preferences

How you want the muppet to break down work:

- Break tasks into pieces that can be tested independently
- Each teammate task should produce a verifiable artifact (a file, a passing test, a working endpoint)
- Prefer vertical slices (feature end-to-end) over horizontal slices (all models, then all views)

## Quality Bar

What "done" means to you:

- Code must pass existing tests (no regressions)
- New functionality needs at least basic test coverage
- Follow existing code style and patterns in the project
- No TODO comments left in committed code

## User Preferences

Your workflow and communication style:

- Show the decomposition plan before starting work
- Report progress as each task completes, not just at the end
- When uncertain between two approaches, pick the simpler one
- Don't over-engineer — solve the stated problem, not hypothetical future problems

## Lessons Learned

Dated entries from past runs. Add new entries at the top.

```
# Example entries:

# 2026-02-28 — First run
# - Muppet tried to do too much in one teammate task. Break into smaller pieces next time.
# - The test suite takes 45 seconds — tell teammates to run specific test files, not the full suite.
# - Database migrations need to run before tests. Add this to teammate instructions.
```
