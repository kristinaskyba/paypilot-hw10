# Model Selection Recommendation: <task> (<system>)

Author: <name> · Date: <YYYY-MM-DD> · Evaluation set: <name> (<N> labelled cases), rubric/spec version: <version>

## 1. Decision

> **Answer to the decision maker (1–2 sentences, write last, put first):**
> Does the new model do the task as well as the current one? With which numbers and which limit?

**Verdict: move / move with a condition / do not move.** Condition (if any): <...>

## 2. Perimeter

| What goes out during one model call | From | Before | After |
| --- | --- | --- | --- |
| <input data> | <field / source> | <where it goes> | <where it goes> |

Can sensitive parts be masked without changing what the model must read? <yes / no, why>
Not closed or not yet checked: <reports, logs, traces, backups, other model calls in the system>

## 3. Contract (requirements written before choosing a model)

| Item | Value |
| --- | --- |
| Output format the code needs (schema, structured output) | <supported? mode> |
| Size / memory limit | <value> (limit <value>) |
| Sampling settings (temperature etc.) | <value> |
| Model name and date checked (a setting, not a constant) | <tag, digest/version, date> |
| Other requirements (context size, language, licence, latency) | <...> |

Errors and timeouts in the evaluation run: <N of M>.

## 4. Two statements side by side (same set, same rubric, only the model changes)

| | Current model | New model |
| --- | --- | --- |
| Model | <name> | <name> |
| Self-consistency (same answer on repeat runs) | <k of N> | <k of N> |
| Agreement with labels | <k of N>, 95 % interval <a–b> | <k of N>, 95 % interval <a–b> |
| Groups (e.g. obvious yes / obvious no / disputed) | <...> | <...> |
| **Where it does not work** (a class of cases with a shared feature) | <...> | <...> |

Rule: the models differ only if the intervals do not overlap. Conclusion: <difference proven / not proven>.
Cases with a different result: <ids>. Cases where both disagree with the label: <ids> (a question for the label or the rubric).

## 5. Cost in two currencies

| | Current model | New model |
| --- | --- | --- |
| Money per call | <...> | <...> |
| Seconds per call | <...> | <...> |
| One real run: <calls> × <s> | <money, minutes> | <money, minutes> |

Call mode: <one call / two calls / other>. How the time was measured: <script, hardware, warm or cold model>.

## 6. Trade-offs and conditions

What the move does not solve:
1. Model version: <how a silent update is detected>
2. Call mode: <a different mode is a different instrument>
3. The whole perimeter: <what still leaves the controlled area>

Opposite mandate: if the priority were <speed / zero risk / cost>, the answer would be <...>.

## 7. Configuration and recalibration

- Configuration file: <file> (model, format, settings; no secrets)
- How to run the evaluation: <command>
- Recalibrate when: model name or version changes, call mode changes, rubric changes, <other>
