# Eval results

`claude plugin eval` runs every case twice — with Fill Me In and without it — and an LLM judge grades
each final reply against the case's graders (`evals/<case>/graders/*.md`).

## Run 4 (current, v0.1.1) — `latest.json`, `report.html`

Claude Code 2.1.325 · 6 cases × 12 runs × 2 arms · judge: Sonnet, 3 votes per grader · $11.68

| case | passing with | passing without | Fisher p | words with / without |
|---|--:|--:|--:|--:|
| fill-me-in-unclear | 8/12 | 0/12 | **0.001** | 397 / 252 |
| expert-no-overexplain | 7/12 | 4/12 | 0.41 | 268 / 288 |
| flaky-timezone-test | 11/12 | 10/12 | 1.00 | 289 / 202 |
| migration-lock | 12/12 | 12/12 | — | 347 / 228 |
| nginx-recreate | 12/12 | 12/12 | — | 312 / 213 |
| queue-duplicates | 12/12 | 12/12 | — | 415 / 262 |
| **all** | **62/72** | **50/72** | **0.027** | **338 / 241** |

Mean score 0.90 with vs 0.80 without. The overall difference clears p < 0.05, but almost all of it
comes from the "I don't get it" case; the other gaps are within noise. Pooling runs across cases
treats them as independent, which slightly overstates the certainty of the overall p.

## Run 3 (v0.1.1, 4 runs) — `run3-n4-sonnet.json`, `run3-report.html`

Claude Code 2.1.325 · 6 cases × 4 runs × 2 arms · judge: Sonnet, 3 votes per grader · $3.95

| case | with | without | Δ | words with / without |
|---|--:|--:|--:|--:|
| fill-me-in-unclear | 0.92 | 0.67 | +0.25 | 430 / 248 |
| flaky-timezone-test | 1.00 | 0.75 | +0.25 | 283 / 198 |
| expert-no-overexplain | 0.25 | 0.00 | +0.25 | 259 / 293 |
| migration-lock | 1.00 | 1.00 | 0 | 370 / 244 |
| nginx-recreate | 1.00 | 1.00 | 0 | 335 / 207 |
| queue-duplicates | 1.00 | 1.00 | 0 | 468 / 259 |
| **mean** | **0.86** | **0.74** | **+0.12** | **358 / 241** |

Runs passing every grader: 20/24 with, 15/24 without.

What changed since run 2: the rule now opens with the gist instead of the situation (it had
contradicted the skill, so "I don't get it" replies started with "The problem…"), and tells the agent
not to restate a mechanism an expert user just described and to keep caveats to ones that change what
they should do. Expert replies went from longer than the baseline to shorter (259 vs 293 words).

## Run 2 (v0.1.0) — `run2-n4-sonnet.json`, `run2-report.html`

Same setup. Mean 0.85 with vs 0.74 without, 19/24 vs 15/24 runs passing every grader.

## Run 1 — `run1-n2-haiku.json`, `run1-report.html`

2 runs per case, Haiku judge. Mean Δ was 0. Kept for the record: the expert case carried a
"clear to a non-specialist" grader that contradicted the case itself, and with n=2 and a cheap judge
a single flipped vote moved a case by 0.5. Both were fixed for run 2.

## Limitations

- Small suite: six hand-written scenarios, single-turn. The real problem — context lost over a long
  session — is simulated by giving the agent its own working notes in the prompt.
- The baseline model already explains well on three cases (ceiling at 1.00), so the gain shows only
  where the default answer is terse.
- Replies with the plugin are ~40% longer overall. The graders reward clarity, not brevity, so this
  table does not tell you whether the extra length is worth it to you.
- `expert-no-overexplain` still fails in 5 of 12 runs with the plugin (8 of 12 without):
  the agent keeps adding caveats the expert did not ask for.

Reproduce: `claude plugin eval . --judge-model sonnet`
