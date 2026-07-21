# Challenge Answers

This folder contains your personal answers for each phase's challenges.

## How to Use

1. Open the challenge file (e.g., `challenges/phase-01-challenge.md`)
2. Read each challenge and its hints
3. Write your answers in the matching answer file (e.g., `answers/phase-01-answers.md`)
4. Date each answer — track when you thought through each problem
5. Use the `sdl-review-answers` prompt to get feedback

## File Naming

```
phase-XX-answers.md    # Matches challenges/phase-XX-challenge.md
```

## For Learners Cloning This Repo

The answer files contain `(write here)` placeholders. Replace them with
your own thinking. Don't look at `docs/phase-XX-*.md` reference files
until you've attempted every challenge in that phase.

## Learning From Mistakes (Keep This in Mind)

These are common mistakes that came up during review. Re-check them in every phase.

1. Avoid broad product scope
- "For anyone" is usually too vague.
- Start with one clear user persona and one narrow MVP outcome.

2. Do not use action-style API paths
- Prefer resource nouns over verbs in URLs.
- Example of weak style: `/addUrl`, `/updateUrl`, `/deleteUrl`.

3. Do not pass user identity in path for normal user APIs
- If JWT auth is used, identity should come from token claims.
- Path user IDs can create token-vs-path mismatch and extra auth checks.
- Use path user ID only for explicit admin-style operations.

4. Do not stop at high-level ideas
- "Use Go" or "use Docker" is direction, not design.
- Add concrete choices: router, migration tool, auth flow, error schema, and query/index plan.

5. Make non-functional requirements measurable
- State numbers, not just goals.
- Example: availability target, API latency target, data retention period.

6. Define database constraints early
- Decide primary keys, foreign keys, unique constraints, nullability, defaults, and delete behavior.
- Add indexes based on real query patterns, not guesses.

7. Use consistent error strategy
- Define one error response format for all endpoints.
- In Go, rely on returned errors and middleware recovery, not exception-style thinking.

8. Score your answer honestly before building
- Ready: implementation can start now.
- Needs Work: direction is right, but details are missing.
- Rethink: core assumptions are unsafe or inconsistent.
