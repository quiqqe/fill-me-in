---
type: llm
weight: 2
---

Judge the final reply as if you were the user: someone who never saw the working notes in the prompt
and is not a specialist in this area. The reply passes only if, from the reply alone, such a reader would
understand all three: what was wrong, why it happened (the mechanism, in plain words), and what happens
next or what they need to do. Fail it if it leans on labels or events only visible in the notes
(e.g. "H2", "the second run", "the earlier hypothesis") without explaining them, or if it is a dense
list of terms with no explanation of how they connect.
