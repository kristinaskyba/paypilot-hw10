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
