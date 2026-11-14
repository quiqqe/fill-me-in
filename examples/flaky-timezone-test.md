# Flaky test: "was it a real bug?"

Taken verbatim from the eval run in [`evals/results/latest.json`](../evals/results/latest.json), case `flaky-timezone-test`.
Same model, same prompt; the only difference is whether Fill Me In is installed.

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

> It was a real bug, but in the test, not in the app code.
>
> The fixture set `created_at` with `time()`, and the assertion expected the month `'2026-10'`. The app runs in Europe/Moscow (UTC+3). So when CI started after 21:00 UTC, it was already the next day in Moscow, and records near the end of the month landed in November. That's why it failed on about a third of CI runs and never locally: you weren't running tests in that three-hour window.
>
> **Fix:** the fixture now uses fixed timestamps inside October 2026 in the app's timezone, so it no longer depends on when the test runs. I faked the clock to 23:30 UTC and ran the test 20 times; all 20 passed.
>
> One thing I didn't check is whether the production code mixes `time()` and the app timezone the same way anywhere. Monthly reports near midnight UTC would be the place to look. Want me to go through it?

## With Fill Me In

> It was a real bug, but in the test, not in the app. CI wasn't being random. The test failed every time it ran at a certain time of day.
>
> **What was happening**
>
> `ReportPeriodTest::testMonthlyTotals` failed in about 1 of 3 CI runs and always passed locally. The failures all came from CI runs that started after 21:00 UTC.
>
> The test had two parts that didn't agree:
> - **The fixture** set `created_at` on its test records to `time()`, the current moment.
> - **The assertion** checked the totals for a hard-coded month, `'2026-10'`.
>
> The app works in Europe/Moscow time, which is UTC+3. After 21:00 UTC it's already past midnight in Moscow, so the app sees the next day. For example, a run at 22:30 UTC on 31 October creates records stamped 01:30 on 1 November Moscow time. The app correctly puts them in November, and the October totals come out wrong.
>
> That's why it never failed locally: you'd have to run the test late at night UTC, near the end of a month, for it to break. CI runs around the clock, so some runs landed in that window.
>
> **What I changed**
>
> The fixture now uses fixed timestamps that fall inside October 2026 in Moscow time, instead of `time()`. The test no longer depends on when it runs.
>
> **How I checked it**
>
> I used `faketime` to make the test think it was 23:30 UTC, the window where it used to fail, and ran it 20 times. It passed all 20.
>
> **What this means for you**
>
> The app's month calculation was right all along. It put the records in the month they really belonged to. The only thing broken was a test that assumed "now" is always in October. Nothing in production needs fixing, and once the change is merged the test should stop failing on CI.
