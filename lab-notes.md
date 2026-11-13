# HW10 lab notes

## Predictions (written before step 3, 2026-11-13)
- Agreement of local judge with my labels: 20 of 34 (cloud Sonnet: 25 of 34)
- Where it misses first: no idea
- Seconds per call: 3

## Local judge contract
- Tag: qwen3:8b, ID 500a1f067a9f, size 5.2 GB (limit 6 GB), checked 2026-11-13
- Ollama 0.40.16 (Homebrew), runs on the Mac, not in Docker
- Self-test: OK, first call 20 s (model loading into memory)

## Step 2: LB-11, 3 runs per mode
| Mode | Valid JSON | Verdicts | Quote found | Time per call |
| --- | --- | --- | --- | --- |
| schema | 3 / 3 | FFF (label FAIL) | 3 / 3 | 8.5 s |
| none | 3 / 3 | FFF (label FAIL) | 3 / 3 | 7.1 s |

## Step 3: calibration on 34 cases (JUDGE_FORMAT=schema, runs 5)
- Report: reports/calibration-l09_labels-20261113T164231.json (judge_model ollama:qwen3:8b)
- Agreement 26/34 = 0.76 (0.60–0.88); self-consistency 33/34 (LB-10 PFFFF)
- Obvious 22/24, disputed 4/10; errors 0; quote found 170/170
- Time 1523 s, 8.96 s per call

## Step 4: two judges side by side
| | Cloud (Sonnet 5.5) | Local (qwen3:8b) |
| --- | --- | --- |
| Agreement | 25/34 (0.57–0.85) | 26/34 (0.60–0.88) |
| Self-consistency | 33/34 | 33/34 |
| Obvious / disputed | 21/24 / 4/10 | 22/24 / 4/10 |
- Ranges overlap: the judges are NOT different on this set.
- Different verdict: only PL-17 (cloud PASS = miss, local FAIL = match).
- Both judges miss the same 8 cases, 5 of 5 times: LB-02, LB-04, LB-12, PL-07, PL-12, PL-14, PL-15, PL-20.
  Same misses on two very different judges point to the labels or rubric, not the judge.

## Step 5: cost in two currencies
| | Cloud | Local |
| --- | --- | --- |
| Money, release run (60 calls) | about USD 0.35–0.55 | 0 |
| Seconds per call | not measured in L09 | 8.96 s |
| Release run, one call / two calls | not measured | 9 min / 18 min |
- Hardware: Mac, 24 GB RAM, Ollama 0.40.16 (Homebrew).

## Predictions vs result
| | Prediction | Result |
| --- | --- | --- |
| Agreement | 20/34 | 26/34 |
| Where it misses | no idea | same 8 cases as cloud |
| Seconds per call | 3 | 8.96 |
