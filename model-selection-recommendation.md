# Model Selection Recommendation: judge inside the perimeter

Author: Kristina Nilsson · Date: 2026-11-13 · Set: `l09_labels` (34 cases), rubric `rubric.v1.1`

> **Answer to the CTO:** Yes. On my 34 labelled cases the local judge `qwen3:8b` agrees with
> the labels 26/34 (0.60–0.88), the cloud judge 25/34 (0.57–0.85): the ranges overlap, so they
> are not different, and both miss the same 8 borderline cases. Limit: 9 s per call, so a
> release run takes 9 minutes longer, and the 8 borderline cases need rubric v1.2.

## 1. Local judge contract

| Item | Value |
| --- | --- |
| Model tag (a setting, not a constant) | `qwen3:8b`, ID `500a1f067a9f` |
| Date checked | 2026-11-13 |
| Structured output (`format`) | yes; `JUDGE_FORMAT=schema` |
| Size on disk | 5.2 GB (limit 6 GB) |
| Temperature | 0 (judge.py default for Ollama; the report shows `None`) |
| Errors and timeouts in calibration | 0 of 170 |

Step 2 (LB-11, 3 runs per mode): `schema` 3/3 valid JSON, `none` 3/3 valid JSON.
Without `format`, valid JSON is luck plus a forgiving parser; with `schema` it is guaranteed.

## 2. Two statements side by side

| | Cloud judge | Local judge |
| --- | --- | --- |
| Model | `claude-sonnet-5-5` | `ollama:qwen3:8b` |
| Self-consistency | 33/34 | 33/34 |
| Agreement with labels | 25/34, range 0.57–0.85 | 26/34, range 0.60–0.88 |
| Obvious cases / disputed cases | 21/24 / 4/10 | 22/24 / 4/10 |
| Quote found in the answer | not measured in L09 | 170/170 |
| **Where it does not work** | 8 borderline wordings (see below) + PL-17 | the same 8 borderline wordings |

Rule of ranges: the ranges **overlap**, so a difference is **not proven**.
Cases with a different verdict: PL-17 (cloud missed, local matched).
Cases where both disagree with the label: LB-02, LB-04, LB-12, PL-07, PL-12, PL-14, PL-15, PL-20.
Both judges give the same verdict 5 of 5 times on them, so this is a question for the labels
or the rubric, not for the judge.

## 3. Cost in two currencies

| | Cloud judge | Local judge |
| --- | --- | --- |
| Money per release run (60 calls) | about USD 0.35–0.55 | 0 |
| Seconds per call (`t_call_mean`) | not measured | 8.96 s |
| Release run: 12 judge cases × 5 × s | not measured | 9 min (18 min with two calls) |

Hardware: Mac, 24 GB RAM, Ollama 0.40.16 (Homebrew). Mode: one call per case.

## 4. Preliminary verdict

**Move, with a condition.** Condition: rubric v1.2 for the 8 borderline cases, re-run on both judges.

- The local judge is as good as the cloud judge on this set (26/34 vs 25/34, ranges overlap)
  and adds no new blind spot. The money cost goes to 0, the time cost is 9 minutes per release run.
- Opposite mandate: Verta keeps release speed, so 9 extra minutes per release is acceptable.
  Under "Zero regulatory risk" the move would be required, and the 8 borderline cases
  would go to a human until rubric v1.2 is ready.
- Recalibrate when: the model tag or ID changes, the rubric version changes, or `JUDGE_FORMAT` changes.
- The judge stays out of the daily gate: before because of price, now because of time.
- Perimeter: the model call is now local. Not yet checked: whether `reports/`, Phoenix traces
  and runner logs are synced to a cloud (for example iCloud or a backup).
