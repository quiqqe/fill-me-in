<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg">
    <img src="assets/logo.svg" width="220" alt="Fill Me In: someone peeking in, having missed everything">
  </picture>
</p>

<h1 align="center">Fill Me In</h1>

<p align="center">
  <em>You walked in halfway through. It catches you up.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/github/stars/quiqqe/fill-me-in?style=flat-square&color=111111&label=stars" alt="Stars">
  <img src="https://img.shields.io/badge/claude%20code-plugin-111111?style=flat-square" alt="Claude Code plugin">
  <img src="https://img.shields.io/github/v/release/quiqqe/fill-me-in?display_name=release&style=flat-square&color=111111&label=release" alt="Release">
  <img src="https://img.shields.io/badge/license-MIT-111111?style=flat-square" alt="MIT license">
</p>

<p align="center">
  <strong>Situation first &middot; what, why, what to do &middot; depth that follows the conversation &middot; outranks "be brief" plugins</strong>
</p>

<p align="center">
  <sub><a href="README.ru.md">Русский</a></sub>
</p>

---

You know the feeling. You hand the agent a task, look away for ten minutes, come back, and read:

> Dropped `set $backend`, moved to an upstream with `zone` and `resolve`, `valid=1s`. Recreating app no longer needs a reload.

Every word is familiar. None of it means anything. The agent was in the room the whole time — it read the files, dropped two hypotheses, named things along the way — and it writes as if you were there too.

Fill Me In makes the agent write for the person who just walked in.

## Before / after

Same fix, with Fill Me In:

> **Gist:** nginx now notices on its own that the app container was recreated, so it no longer needs a restart.
>
> **How it works.** nginx reaches the app container by the name `app`, and Docker's DNS turns that name into an IP. Recreate the container and the IP can change; an nginx that remembered the old one sends requests into the void. The new `upstream` block with `resolve` makes nginx re-ask DNS every second (`valid=1s`).
>
> ```
> request → nginx → upstream backend → DNS: "app = ?" (every 1 s) → app container
> ```
>
> **Example.** A deploy recreates app, its IP goes from .5 to .7, and within a second nginx sends traffic to .7. Before, you would restart nginx by hand.

Not longer for the sake of it. Longer by exactly the context you missed. Real side-by-side replies from the eval run: [examples/](examples/).

## Numbers

Six scenarios where the agent reports on work you didn't see, each run 4 times with and without the plugin, graded by a Sonnet judge on one question: *would someone who never saw the work understand what was wrong, why, and what happens next?*

| | with | without |
|---|--:|--:|
| mean score | **0.85** | 0.74 |
| runs passing every grader | **19/24** | 15/24 |
| words per reply | 380 | 251 |

The gain sits where the default answer is terse (a flaky test, an "I don't get it" follow-up); on three cases the model already explains well and both arms score 1.00. Replies get ~50% longer. One honest miss: when the user is the expert, both arms still pile on caveats. Small suite, single-turn — method, per-case table and limitations in [evals/results](evals/results/README.md). Reproduce with `claude plugin eval . --judge-model sonnet`.

## How it works

Two parts, one plugin.

**An always-on rule.** A `SessionStart` hook (also after `/clear` and context compaction) loads it, and a one-line reminder rides along with every prompt so other instructions don't drown it out:

```
1. Situation first      → what was going on, what the agent found
2. Then the conclusion  → what, why, what it means for you
3. No private context   → "that bug", "the second hypothesis" get explained or cut
4. Concrete over vague  → "request X returns Y because Z"
5. Depth follows you    → per topic, from what you say, not what feels obvious to the agent
```

Depth is the part agents usually get wrong, so it is read from the dialogue instead of guessed:

| You… | The agent… |
|---|---|
| use the topic's terms, correct it on substance | skips the basics |
| ask what a term means, say "I don't get it" | explains the mechanism and stays at that level for the topic |
| give no signal | explains — an extra sentence is cheaper than an answer you can't follow |

**`/fill-me-in` for when an answer still lands badly.** The agent finds the invisible context its answer leaned on, re-checks key facts against the source instead of memory, and re-explains: the gist in one line, the mechanism, a step-by-step example, what follows. `/fill-me-in <part>` unpacks just that part.

## Plays well with "be brief"

Running [ponytail](https://github.com/DietrichGebert/ponytail), [caveman](https://github.com/JuliusBrussee/caveman) or your own "max three lines" rule? Keep them. Fill Me In tells the agent to let them cut code, repetition and pleasantries — never context, reasons or the mechanism. You installed it to understand answers, so on explanations it wins. A direct request in your message ("code only") still wins over both.

## Install

### Claude Code

```
/plugin marketplace add quiqqe/fill-me-in
/plugin install fill-me-in@quiqqe
```

Starts working from the next session.

### Just the skill, any agent

`/fill-me-in` alone, without the always-on rule — for agents that support `SKILL.md`:

```bash
npx skills add quiqqe/fill-me-in
```

### Uninstall

```
/plugin uninstall fill-me-in@quiqqe
```

<details>
<summary><strong>Why a plugin and not just a skill?</strong></summary>

A skill loads only when the agent decides the task matches it. "Write clearly" applies to every answer, so as a skill it would misfire exactly on the quick replies where context goes missing. A plugin can load the rule on every session start through a hook, and still ship `/fill-me-in` as the fallback.

</details>

## License

MIT
