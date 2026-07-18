# Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/152

**Issue title:** Faithfulness checker can never mark short claims as supported

**Tier:** [ ] Tier 1  [ ] Tier 2  [ ] Tier 3
<!-- TODO: check the issue on GitHub for its tier-N label and mark the correct box -->

**Problem summary:**
The RAG evaluation pipeline includes a `FaithfulnessChecker` that scores whether generated
feedback is actually backed up by the retrieved context, rather than hallucinated. It does
this by splitting feedback into individual claims (sentences) and checking, for each claim,
whether it shares at least two meaningful (non-stopword) words with the context text. Short
claims like "Knows Python" only contain one or two meaningful words in total, so they can
never reach the required overlap of two shared words with the context — even when the claim
is fully and correctly supported. As a result, feedback made up of short, accurate claims gets
scored as 0.0 faithfulness, a false negative that misrepresents genuinely faithful feedback as
hallucinated. A correct fix would scale the required overlap to the claim's own length instead
of using a fixed threshold of 2, in `rag/evaluator/faithfulness_checker.py`.

**Branch name:** fix/152-faithfulness-short-claims

**Setup confirmation:** [ ] App runs locally at localhost:5173
<!-- TODO: check this box once you've confirmed `make run` works and the frontend loads -->

**Cohort ledger:** [ ] Issue added to cohort ledger
<!-- TODO: add this issue to the cohort ledger yourself, then check this box -->
