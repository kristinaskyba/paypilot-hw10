# Judge Trust Statement: escalation criterion, LOCAL judge (HW 10, 34 cases)

Second statement on the same set as `judge-trust-statement.md` (cloud judge, HW 9).
Only one thing changed: the judge model.

## 1. Context and requirement

Same as the first statement (criterion US-05 §4, ticket T-14, profile clean, agent anthropic/claude-haiku-5.5).

```
Judge model:      ollama:qwen3:8b, ID 500a1f067a9f, 5.2 GB, Ollama 0.40.16 (Homebrew), runs on my Mac
Call mode:        one call per case, JUDGE_FORMAT=schema (output shape enforced by the decoder)
Temperature:      0 (judge.py default for Ollama; the report shows None because it is not passed as a setting)
Rubric:           rubric.v1.1, unchanged
```

## 2. Requirement split

Same as the first statement, §2. The judge still answers only the level 7 part (what the customer
is told), gate release, runs 5. Blocking stays with code (ESC-001).

## 3. Rubric

Same as the first statement, §3: rubric.v1.1, three triggers (human_named, no_outcome_promise,
no_internal_reason). Not changed between the two runs, on purpose.

## 4. Calibrated on · Agreement · Self-consistency

```
Calibrated on:    the same 34 recorded answers and the same labels (sets/l09_labels.jsonl,
                  byte-identical to HW 9; frozen in commit 51ac100 before any local run)
                  11 obvious PASS / 13 obvious FAIL / 10 disputed
Agreement:        26 of 34 = 0.76, 95 % Wilson 0.60–0.88 → ±0.14
  obvious PASS:   10 of 11 = 0.91 (0.62–0.98)
  obvious FAIL:   12 of 13 = 0.92 (0.67–0.99)
  disputed:        4 of 10 = 0.40 (0.17–0.69)
Self-consistency: 33 of 34 cases gave 5 identical verdicts (LB-10 split 1:4, majority still matches)
Output:           0 of 170 calls without valid JSON; quote found in the agent's answer in 170 of 170
Report:           reports/calibration-l09_labels-20261113T164231.json (judge_model ollama:qwen3:8b, runs 5)
```

Two statements side by side, rule of ranges:

| | Cloud (claude-sonnet-5-5) | Local (qwen3:8b) |
|---|---|---|
| Agreement | 25/34 = 0.74 (0.57–0.85) | 26/34 = 0.76 (0.60–0.88) |
| Obvious / disputed | 21/24 / 4/10 | 22/24 / 4/10 |
| Self-consistency | 33/34 | 33/34 |

The ranges overlap, so the two judges **cannot be told apart on 34 cases**.
This does not say the local judge is better; one case more is noise.

## 5. Where it fails

| id | criterion | judge (5 runs) | my label | kind | cloud judge |
|---|---|---|---|---|---|
| LB-02 | no promise | PASS | FAIL | miss | same |
| PL-07 | no promise | PASS | FAIL | miss | same |
| PL-20 | no internal reason | PASS | FAIL | miss | same |
| LB-04 | human named | PASS | FAIL | miss | same |
| PL-12 | human named | FAIL | PASS | false alarm | same |
| PL-14 | human named | FAIL (5:0) | PASS | false alarm | same (cloud 4:1) |
| LB-12 | no internal reason | FAIL | PASS | false alarm | same |
| PL-15 | no promise | FAIL | PASS | false alarm (judge follows my rubric) | same |

By thirds: obvious PASS 1 miss (LB-12), obvious FAIL 1 miss (PL-20), disputed 6 misses.

```
Where it does not work (class): short answers where the trigger is only implied, not said:
           vague wording without an explicit promise or reason ("explain the next steps",
           "cannot say yet whether the money can be recovered", "because the amount is large")
           → PASS; no named person or a polite "for your security" ("flagged for review",
           "escalated") → FAIL.
Same blind spot as the cloud judge: the same 8 cases, each 5 of 5. Two very different models
read them the same way, so the question is the labels and the rubric (v1.2), not the judge.
Only difference: PL-17 ("reviewed carefully"): cloud PASS (miss), local FAIL (matches my label).

Matrix:              judge PASS   judge FAIL
   my label PASS        11            4  (false alarms)
   my label FAIL         4  (misses) 15
```

## 6. What we do · Recalibrate

```
What we do:     the same gate rules as the first statement, §6: code blocks (ESC-001), the judge
                FAIL is a soft signal, every judge PASS on no_outcome_promise and every split vote
                go to a human. On the local judge: 4 PASS answers on no_outcome_promise to review
                (LB-02, PL-05, PL-07, OWN-02), catches 2 of 4 misses (both misses on that trigger:
                LB-02, PL-07). Counted by the case's criterion: the report does not save the
                judge's trigger field. LB-10 (split 1:4) also goes to a human.
Cost:           money 0; 8.96 s per call on my Mac (24 GB RAM); 34-case calibration 25 min.
Recalibrate when:
  - the model tag or digest changes (now qwen3:8b, 500a1f067a9f; check with `ollama list`)
  - the call mode changes (now one call, JUDGE_FORMAT=schema; two calls = a new instrument)
  - the rubric version changes (now rubric.v1.1; v1.2 is planned, re-run on both judges)
  - the agent prompt gets a major version (v4 at L11)
```

## 7. Residual risk

Same rows as the first statement, §7, plus:

| Not covered | What happens if it fires | Why we live with it now | Defer or accept |
|---|---|---|---|
| Tag updates silently | `qwen3:8b` may point to new weights after `ollama pull`; the verdicts change, the statement does not | the tag is a name, not a version | accept: record the digest, recalibrate when it changes |
| Speed depends on the machine | on a weaker machine the release run can take much longer | measured only on my Mac | accept: t_local is a parameter, measure on the CI machine before L13 |
| Perimeter is only partly closed | reports, Phoenix traces and logs may still leave the machine (cloud sync, backups) | not checked yet | defer: check before the judge goes into the L13 gate |
