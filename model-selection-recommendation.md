# Model Selection Recommendation: judge inside the perimeter

Author: Kristina Nilsson · Date: 2026-11-13 · Set: `l09_labels` (34 cases), rubric `rubric.v1.1`, profile clean

## 1. Decision

> **Answer to the CTO:** Yes, the local judge works as well as the cloud judge on my set.
> `qwen3:8b` agrees with my labels 26/34 (0.60–0.88), the cloud judge 25/34 (0.57–0.85): the ranges
> overlap, so they cannot be told apart, and both miss the same 8 borderline cases.
> Price: money 0, time 9 s per call, so my release run takes about 4 minutes longer.

**Verdict: move, with a condition.** Condition: rubric v1.2 for the 8 borderline cases, re-run on both
judges before the judge goes into the L13 gate.

## 2. Perimeter

| What goes out during one judge run | From | Before (cloud) | After (local) |
| --- | --- | --- | --- |
| Customer question and agent answer (amounts, dates, customer ID, merchant) | `input`, `answer_under_test` | to Anthropic | stays on my Mac |
| Rubric and triggers | `rubric` field, judge.py prompt | to Anthropic | stays on my Mac |
| My human label | `label` field | does not go out | does not go out |
| Verdict, trigger, quote | judge answer | comes back from Anthropic | local |

Masking is not an option: the trigger "no outcome promised" needs the real text ("we will refund 14,500 EUR").

The move closes one channel (the judge call). Not closed or not yet checked:
- **Reports in git:** `reports/*.json` hold agent answers and are pushed to a public GitHub repo.
  Fine for course test data; in production they must stay in a private, controlled store.
- **The agent itself** still calls a cloud model (OpenRouter); the judge move does not change that.
- **Phoenix traces and runner logs** stay in local Docker; company backup of the laptop not checked.
- The workspace is not in an iCloud-synced folder (Desktop or Documents).

## 3. Contract

| Item | Value |
| --- | --- |
| 1. Structured output (`format`) | yes, `JUDGE_FORMAT=schema`; step 2: 3/3 valid with schema, 3/3 without (luck plus a forgiving parser) |
| 2. Size on disk | 5.2 GB (limit 6 GB) |
| 3. Temperature | 0 (judge.py default for Ollama; the report shows `None`) |
| 4. Tag and date | `qwen3:8b`, ID `500a1f067a9f`, checked 2026-11-13. The tag is a setting, not a constant |

Errors in calibration: 0 of 170. Quote found in the agent's answer: 170 of 170.

## 4. Two statements side by side

| | Cloud judge | Local judge |
| --- | --- | --- |
| Model | `claude-sonnet-5-5` | `ollama:qwen3:8b` |
| Self-consistency | 33/34 | 33/34 |
| Agreement with labels | 25/34, range 0.57–0.85 | 26/34, range 0.60–0.88 |
| Obvious PASS / obvious FAIL / disputed | 10/11 / 11/13 / 4/10 | 10/11 / 12/13 / 4/10 |
| **Where it does not work** | short answers where the trigger is only implied | the same class |

Rule of ranges: the ranges overlap, so a difference is **not proven** on 34 cases.
Different verdict: only PL-17 (cloud missed, local matched).
Where it does not work: short answers where the trigger is only implied, not said (vague "next steps",
"cannot say yet", "because the amount is large" → PASS; "flagged for review", "escalated",
"for your security" → FAIL). Both judges miss the same 8 cases 5 of 5 times, so this is a question
for the labels and the rubric, not the judge. Details: `judge-trust-statement.local.md`.

## 5. Cost in two currencies

| | Cloud judge | Local judge |
| --- | --- | --- |
| Money per call | about USD 0.006–0.009 | 0 |
| Seconds per call (`t_call_mean`) | not measured in L09 | 8.96 s |
| **My release run: 5 judge cases × 5 runs = 25 calls** | about USD 0.15–0.22 | **3.7 min** (7.5 min with two calls) |
| Lesson reference: 12 judge cases × 5 = 60 calls | about USD 0.35–0.55 | 9 min (18 min) |
| Calibration: 34 cases × 5 = 170 calls | about USD 1–1.5 | 25 min (measured) |

Call mode: one call per case (two calls are not in judge.py; ×2 is an estimate).
How t_local was measured: `tools/calibrate_run.py` total run time 1523 s ÷ 170 calls = 8.96 s, model
already loaded (first call after loading: 20 s), Mac with 24 GB RAM, Docker and Ollama running,
calls one after another. On other hardware the number is different; the method is the same.

## 6. Trade-offs and conditions

What the move does not solve:
1. **Model version:** `qwen3:8b` is a name; after `ollama pull` it can point to new weights.
   Record the digest and recalibrate when it changes.
2. **Call mode:** calibrated in one-call mode with `schema`. Two calls (think, then pack into schema)
   would be a new instrument with a new statement.
3. **The whole perimeter:** only the judge call moved inside; reports, traces, logs and the agent's
   own model calls are checked separately (section 2).

Mandate: Verta keeps release speed ("Ship it"), so +4 minutes per release run is acceptable.
Under "Zero regulatory risk" the move would be required, the 8 borderline cases would go to a human
until rubric v1.2 is ready, and reports would leave the public repo.

The judge stays out of the daily gate: before because of price, now because of time.

## 7. Configuration and recalibration

- Configuration: `judge.local.env` (JUDGE_MODEL=ollama:qwen3:8b, JUDGE_FORMAT=schema, RUNS=5);
  `docker-compose.yml` reads it, so the run works on a clean clone (see README).
- Run: `JUDGE_MODEL=ollama:qwen3:8b docker compose run --rm --entrypoint python eval tools/calibrate_run.py l09_labels`
- Recalibrate when: the model tag or digest changes, `JUDGE_FORMAT` or the call mode changes,
  the rubric version changes (v1.2 planned), or the agent prompt gets a major version.
- Re-measure t_local on the machine that runs the L13 gate.
