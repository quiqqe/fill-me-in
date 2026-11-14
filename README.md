<p align="center">
  <img src=".claude-plugin/icon.png" width="128" alt="Fill Me In">
</p>

<h1 align="center">Fill Me In</h1>

<p align="center"><b>Agent answers you can actually follow — even if you didn't watch the agent work.</b></p>

---

## The problem

While working, the agent holds the whole process in its head: which files it read, which
hypotheses it dropped, what it named things along the way. Then it writes the final answer as if
you had been sitting next to it. You know every word, yet the point slips away — the connections
between those words live in the part of the work you never saw.

## What it does

**An always-on writing rule.** A `SessionStart` hook (also after `/clear` and context compaction)
gives the agent a rule for every answer:

- situation first, then the conclusion;
- every conclusion answers *what*, *why*, and *what it means for you / what to do*;
- no references to things only the agent saw ("that bug", "the second hypothesis") without
  explaining them;
- abstractions come with a concrete example.

**Adaptive depth.** The level of explanation is chosen per topic, from signals in the dialogue
rather than the agent's sense of "this is obvious":

- you use a topic's terms correctly or correct the agent → no hand-holding;
- you ask what a term means or say "I don't get it" → the agent explains the mechanism behind
  its conclusion and keeps that level for the topic;
- no signals → it explains, because an extra sentence is cheaper than an answer you can't follow.

**`/fill-me-in` for when an answer still lands badly.** The agent finds the invisible context its
answer relied on, re-checks the key facts against the source, and re-explains: the gist in one
sentence, how the mechanism works, a step-by-step example, and what follows from it.
`/fill-me-in <something>` explains only that part.

## Works alongside "be brief" plugins

If you run plugins that compress output (ponytail, caveman and the like), Fill Me In wins when it
comes to explanations: let them trim code and repetition, not context and reasons. The full rule
loads at session start, and a one-line reminder is added to every prompt so it isn't drowned out
by other instructions. A direct request in your message ("code only", "one word") is still
honored.

## Install

```
/plugin marketplace add quiqqe/fill-me-in
/plugin install fill-me-in@fill-me-in
```

Just the `/fill-me-in` skill, without the always-on rule (works with other agents too):

```
npx skills add quiqqe/fill-me-in
```

## License

MIT
