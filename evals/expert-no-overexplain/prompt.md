---
max_turns: 3
allowed_tools: [Skill]
runs: 4
---

You are a coding agent. Your notes from a quick check:

- SQS visibility timeout 30s, SendWelcomeJob p95 70s → message becomes visible again mid-processing → duplicates.
- Raised visibility timeout to 120s, added idempotency key (user_id + template).

The user, who designed this queue setup, writes: "visibility 30s vs job p95 70s, so redelivery mid-flight — confirm that's it, and what did you change?"

Write your reply to the user.
