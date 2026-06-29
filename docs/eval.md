# Evaluation

How we measure whether the app's answers are actually good — not just "looked right once."

## Our question set

Start from the ready-made set in [`../sample-data/README.md`](../sample-data/README.md), then add questions specific to your real corpus. Include **2–3 questions the documents do NOT cover**, so you can confirm the app says "I don't know" instead of inventing an answer.

| # | Question | Expected answer | In scope? |
|---|----------|-----------------|-----------|
| 1 | _e.g. How much does Nimbus Pro cost?_ | _$8/month_ | Yes |
| 2 | | | Yes |
| 3 | | | Yes |
| … | | | |
| n | _an out-of-scope question_ | _(should say "I don't know")_ | **No** |

## Scoring

For each question, mark the answer **correct / partial / wrong / hallucinated**, and whether it **cited the right source**. A quick way to get a single number: count `correct` out of total.

## Run log

Record every run so you can see whether a change actually helped. Re-run the whole set after changing chunk size, top-k, the embedding model, or the prompt.

| Run date | Model | Chunk size | Overlap | Top-k | Score (/10) | Notes |
|----------|-------|-----------|---------|-------|-------------|-------|
| 2026-07-15 | llama3.1:8b | 1000 | 0 | 3 | 6 | baseline |
| 2026-07-16 | llama3.1:8b | 400 | 50 | 4 | 9 | smaller chunks fixed retrieval misses |

> The story this table tells — "we changed X and the score moved from A to B" — is one of the best things you can show in your demo and talk about in an interview.
