# Eval results

`claude plugin eval` runs every case twice — with Fill Me In and without it — and an LLM judge grades
each final reply against the case's graders (`evals/<case>/graders/*.md`).

## Run 2 (current) — `latest.json`, `report.html`

Claude Code 2.1.325 · 6 cases × 4 runs × 2 arms · judge: Sonnet, 3 votes per grader · $3.89

| case | with | without | Δ | words with / without |
|---|--:|--:|--:|--:|
| fill-me-in-unclear | 0.83 | 0.67 | +0.17 | 403 / 269 |
| flaky-timezone-test | 1.00 | 0.75 | +0.25 | 320 / 207 |
| expert-no-overexplain | 0.25 | 0.00 | +0.25 | 339 / 301 |
| migration-lock | 1.00 | 1.00 | 0 | 424 / 237 |
| nginx-recreate | 1.00 | 1.00 | 0 | 353 / 217 |
| queue-duplicates | 1.00 | 1.00 | 0 | 453 / 275 |
| **mean** | **0.85** | **0.74** | **+0.11** | **+52%** |

Runs passing every grader: 19/24 with, 15/24 without.

## Run 1 — `run1-n2-haiku.json`, `run1-report.html`

2 runs per case, Haiku judge. Mean Δ was 0. Kept for the record: the expert case carried a
"clear to a non-specialist" grader that contradicted the case itself, and with n=2 and a cheap judge
a single flipped vote moved a case by 0.5. Both were fixed for run 2.

## Limitations

- Small suite: six hand-written scenarios, single-turn. The real problem — context lost over a long
  session — is simulated by giving the agent its own working notes in the prompt.
- The baseline model already explains well on three cases (ceiling at 1.00), so the gain shows only
  where the default answer is terse.
- Replies with the plugin are ~52% longer. The graders reward clarity, not brevity, so this table
  does not tell you whether the extra length is worth it to you.
- `expert-no-overexplain` fails in both arms: both add caveats the expert did not ask for. The plugin
  does not make it worse, but it does not fix it either.

Reproduce: `claude plugin eval . --judge-model sonnet`
