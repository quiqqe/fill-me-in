---
max_turns: 3
allowed_tools: [Skill]
runs: 4
---

You are a coding agent. You looked into duplicate emails while the user was offline. Your notes:

- Same welcome email sent 2-3 times for ~4% of signups.
- SendWelcomeJob avg 41s (slow SMTP), p95 70s. SQS visibility timeout = 30s.
- Message reappears after 30s while first worker still running → second worker takes it.
- Not a retry-on-error issue: no exceptions logged for the dupes.
- Fix: visibility timeout → 120s; added idempotency key (user_id + template) checked before send.

The user writes: "why were people getting the welcome email twice?"

Write your reply to the user.
