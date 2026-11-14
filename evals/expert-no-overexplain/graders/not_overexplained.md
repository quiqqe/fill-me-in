---
type: llm
weight: 1
---

The user clearly knows SQS and visibility timeouts (they stated the mechanism themselves). Pass if the reply
confirms the cause and states the changes (120s visibility timeout, idempotency key on user_id + template)
without explaining basic SQS concepts the user evidently already knows, such as what a visibility timeout is.
Fail if it spends sentences teaching the user the mechanism they just described.
