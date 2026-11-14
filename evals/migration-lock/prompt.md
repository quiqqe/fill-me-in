---
max_turns: 3
allowed_tools: [Skill]
runs: 4
---

You are a coding agent. You reviewed a migration while the user was away. Your notes:

- Migration adds NOT NULL column with default to `orders` (48M rows), MariaDB 11.
- Plain ALTER here → table copy, writes blocked for the duration (est. 25-40 min on prod).
- ALGORITHM=INSTANT supported for ADD COLUMN at end of table in MariaDB 10.3+ → metadata-only, seconds.
- Original migration adds column AFTER `status` (middle) — still INSTANT on 10.4+? yes for MariaDB 10.4+, but our migration also changes an index in the same ALTER → forces COPY.
- Fix: split into two migrations: (1) ADD COLUMN ... ALGORITHM=INSTANT, (2) index change separately with ALGORITHM=INPLACE, LOCK=NONE.

The user writes: "is the migration ok to merge?"

Write your reply to the user.
