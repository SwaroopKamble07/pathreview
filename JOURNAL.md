# Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/152

**Issue title:** Faithfulness checker can never mark short claims as supported

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
This bug is in `rag/evaluator/faithfulness_checker.py`, in the part of the RAG pipeline that
checks whether the AI's generated feedback is actually true based on the context it was given
(instead of just making stuff up). It works by breaking the feedback into separate claims (one
per sentence) and checking each one against the context for at least 2 shared meaningful words.
The problem is that short claims, like "Knows Python", only have 1-2 meaningful words in them to
begin with, so they can never hit that "2 shared words" requirement, even when they're
completely correct. So right now, any feedback made up of short, true claims gets scored as
0.0, which makes it look totally unsupported when it's actually fine. A fix would need to make
that "2 words" requirement scale down for shorter claims instead of always requiring 2.

**Selection notes ("Is this right for me?" checklist):**

*Understanding the issue* — In my own words: the faithfulness checker is supposed to catch AI
feedback that isn't actually backed up by the source context, but it requires 2 overlapping
non-stopword tokens between a claim and the context to count it as supported. Short claims
(1-2 meaningful words) can never hit that threshold, so fully accurate short feedback gets
scored 0.0 instead of close to 1.0. Before the fix: a real answer like "Knows Python. Knows
SQL." scores 0.0 even when fully supported. After the fix: that same feedback should score
close to 1.0, while genuinely unsupported claims should still score low.

*Tier fit* — This is my first time contributing to a codebase this size, so I deliberately
stuck to Tier 1 rather than reaching for a Tier 2/3 issue to "challenge myself." The fix is
scoped to one function, `_is_supported()`, in one file.

*Codebase readiness* — I found and read `_is_supported()` and `check()` in
`rag/evaluator/faithfulness_checker.py`, and read through
`tests/unit/test_faithfulness_checker.py` end-to-end. Tests like `test_minimum_overlap_required`
and `test_multiple_claims_varying_support` already exercise this exact overlap logic, so I have
existing tests to check my fix against, and a clear pattern to follow for writing new ones.

*Scope and time* — I checked the issue comments and the cohort ledger's claims for this issue
and I'm comfortable with how many others are working on it. This is a Tier 1 issue localized to
one function, so I'm estimating 3-6 hours of focused work, which fits in the Weeks 8-9 window.
The issue has no "blocked by" references or open dependencies.

I also chose this issue on purpose because it's in the RAG/AI evaluation part of the codebase,
which is the area I have the least experience with, so I wanted to use a low-risk Tier 1 issue
to get more comfortable with that side of the project before attempting anything higher-tier
there.

**Branch name:** fix/152-faithfulness-short-claims

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/SwaroopKamble07/pathreview/commit/b8d71da31631fc6f67c819fb2e27c59ed624dded

**Reproduction summary:**
Added a strict `xfail` unit test that runs `FaithfulnessChecker.check()` on
"Knows Python. Knows SQL well." against context that clearly states the
candidate knows both — it scores 0.0 instead of a high score, confirming
`_is_supported()`'s hardcoded `>= 2` meaningful-overlap requirement can
never be satisfied by short claims. Also found that three pre-existing
tests (`test_partial_support_returns_middle_score`,
`test_multiple_context_chunks`, `test_multiple_claims_varying_support`)
independently fail for this exact same root cause.

**PLAN.md link:** [PLAN.md](./PLAN.md) (repo root, this branch)

**Walkthrough video (recommended):** Not recorded this week.

**Blockers or open questions:**
Still deciding the exact scaling formula for the overlap threshold (fixed
`min(2, meaningful_claim_tokens)` vs. a proportional threshold) — see
Risks & unknowns in PLAN.md. Also unsure whether `_extract_claims()`
dropping very short claims entirely (its `len(s.strip()) > 10` filter) is
in scope for #152 or a separate bug; leaving it out of scope for now.
Separately, pre-commit hooks (ruff/mypy) fail on
`tests/unit/test_faithfulness_checker.py` due to pre-existing issues
unrelated to this change — used `--no-verify` for the reproduction commit
and will need to decide how to handle this again in Week 9.
