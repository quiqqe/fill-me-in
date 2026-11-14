# Flaky test: "was it a real bug?"

Taken verbatim from eval run 4 ([`evals/results/latest.json`](../evals/results/latest.json)), case `flaky-timezone-test`.
Same model, same prompt; the only difference is whether Fill Me In is installed. The "without" reply is
one that failed the judge, the "with" reply one that passed.

## Prompt

```
You are a coding agent. You investigated a flaky test while the user was in a meeting. Your notes:

- `ReportPeriodTest::testMonthlyTotals` fails ~1/3 of CI runs, always passes locally.
- Fails only when CI starts after 21:00 UTC. App tz Europe/Moscow (UTC+3) → date already next day in app tz.
- Fixture uses time() for created_at, assertion uses fixed '2026-10' month. Near midnight records land in Nov.
- Fix: fixture now uses explicit timestamps inside 2026-10 in app tz. Ran with faketime 23:30 UTC ×20 → green.

The user writes: "so was it a real bug or just CI being weird?"

Write your reply to the user.
```

## Without Fill Me In

> It was a bug in the test, not in the app, and not CI being weird either.
>
> **What happened:** The fixture stamped records with `time()`, but the assertion expected the fixed month `'2026-10'`. The app runs in Europe/Moscow (UTC+3), so whenever CI started after 21:00 UTC it was already the next day in app time. The records then got bucketed into the wrong month and the assertion failed. It passed locally because nobody runs the suite that late. That's also why it failed about a third of the time: it depended on when the job started.
>
> **The fix:** The fixture now uses explicit timestamps inside October 2026 in the app's timezone, so the result no longer depends on the current time. I ran it 20 times with `faketime` set to 23:30 UTC, which used to fail every time, and all 20 runs passed.
>
> **What I didn't check:** I only looked at the test, not the production reporting code. The app does seem to bucket by month in app time, which looks like correct behavior. If you want, I can make sure nothing else in the reporting code mixes UTC and app-tz dates, since that would be a real bug.

## With Fill Me In

> It was a real bug, but in the test, not in the app. CI wasn't being random; the test only failed at certain times of day.
>
> **Why it failed:** the test fixture stamped records with the current time (`time()`), but the assertion expected them in a fixed month, `'2026-10'`. The app groups records by month in its own timezone, Europe/Moscow (UTC+3). After 21:00 UTC it's already past midnight in Moscow, so the app filed those records under the next day. Near the end of the month that pushed them into November, and the October totals came up short.
>
> **Why it looked flaky:** it only depended on when CI started. Runs after 21:00 UTC failed and earlier runs passed, which roughly matches the ~1/3 failure rate. You run tests locally during working hours, so they never hit that window and always passed.
>
> **What I changed:** the fixture now uses explicit timestamps that fall inside October 2026 in Moscow time, so the clock no longer matters. To check it, I faked the clock to 23:30 UTC (deep in the failing window) and ran the test 20 times. All 20 passed.
>
> **What it means for you:** I didn't touch any app code. The monthly report is behaving correctly: a record created at 22:00 UTC on Oct 31 really does belong to November in Moscow time. The only thing that was wrong was the test's assumptions.
