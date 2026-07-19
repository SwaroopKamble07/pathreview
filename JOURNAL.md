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

**Branch name:** fix/152-faithfulness-short-claims

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger
