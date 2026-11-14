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
  <img src="https://img.shields.io/badge/eval-62%2F72%20vs%2050%2F72-111111?style=flat-square" alt="Eval: 62/72 clear with the plugin vs 50/72 without">
  <img src="https://img.shields.io/github/v/release/quiqqe/fill-me-in?display_name=release&style=flat-square&color=111111&label=release" alt="Release">
  <img src="https://img.shields.io/badge/license-MIT-111111?style=flat-square" alt="MIT license">
</p>

<p align="center">
  <strong>Gist first &middot; what, why, what to do &middot; depth that follows the conversation &middot; <code>/fill-me-in</code> when an answer doesn't land</strong>
</p>

<p align="center">
  <sub><a href="README.ru.md">Русский</a></sub>
</p>

---

You know the feeling. The agent reports back on something it just did:

> Fixed: dropped `set $php_fpm`, moved to upstream with zone + resolve, valid=1s. Recreating php no longer needs a reload.

Every word is familiar. None of it means anything. The agent was in the room the whole time — it read the files, dropped two hypotheses, named things along the way — and it writes as if you were there too. So you ask "I don't get it", and get the same thing again, a bit longer.

Fill Me In makes the agent write for the person who just walked in.

## Before / after

You reply "I don't get it. What does that mean?" to the message above. Real replies from the eval run, same model, same prompt (opening lines; full text in [examples/](examples/fill-me-in-unclear.md)):

**Without Fill Me In** — the point is buried under an apology and a problem statement:

> Sorry, that was too terse. Here it is in plain terms.
>
> **The problem:** nginx forwards PHP requests to your `php` container by its name, `php`. When the container is recreated (after a rebuild or a `docker compose up` that replaces it), Docker often gives it a new IP address. Out of the box, nginx looks up the name once at startup and keeps using that IP…

**With Fill Me In** — the gist first, then the mechanism with concrete addresses:

> **In short:** nginx used to lose track of your PHP container whenever that container was recreated. I changed the config so nginx re-checks the container's address on its own, so you no longer have to reload nginx after recreating PHP.
>
> **What was going wrong.** nginx passes PHP requests to the PHP container by name, something like `php:9000`. Docker turns that name into an IP address, for example `172.18.0.5`. By default nginx does that lookup once, at startup, and keeps using that IP. When you recreate the PHP container, it usually comes back with a new IP, say `172.18.0.7`…

In this situation the judge rated **0 of 12** replies clear without the plugin and **8 of 12** with it.

## Numbers

Six scenarios where the agent reports on work you didn't see, each run 12 times with and without the plugin. A Sonnet judge grades every reply on one question: *would someone who never saw the work understand what was wrong, why, and what happens next?*

| situation | clear with | clear without | chance it's luck |
|---|--:|--:|--:|
| "I don't get it" after a terse answer | **8/12** | 0/12 | **0.1%** |
| the user is the expert (don't over-explain) | 7/12 | 4/12 | 41% |
| flaky test: "was it a real bug?" | 11/12 | 10/12 | — |
| migration, nginx, queue reports | 36/36 | 36/36 | — |
| **all** | **62/72** (86%) | 50/72 (69%) | **2.7%** |

What this does and doesn't show:

- **Proven:** when an answer didn't land and you ask again, the plugin turns a re-worded repeat into an actual explanation.
- **Not proven:** that first reports get clearer. On these scenarios the model already explains well without it, and the remaining gaps are within noise.
- **Not tested:** the priority over "be brief" plugins below, and real multi-hour sessions — the eval is single-turn and simulates lost context with the agent's own notes.
- **Cost:** replies are ~40% longer (338 vs 241 words).

Method, per-case results and limitations: [evals/results](evals/results/README.md). Reproduce: `claude plugin eval . --runs 12 --judge-model sonnet` (about $12).

## How it works

Two parts, one plugin.

**An always-on rule.** A `SessionStart` hook (also after `/clear` and context compaction) loads it, and a one-line reminder rides along with every prompt so other instructions don't drown it out:

```
1. Gist first           → one plain sentence: what happened
2. Then the situation   → what was going on, what the agent found
3. Every conclusion     → what, why, what it means for you
4. No private context   → "that bug", "the second hypothesis" get explained or cut
5. Concrete over vague  → "request X returns Y because Z"
6. Depth follows you    → per topic, from what you say, not what feels obvious to the agent
```

Depth is the part agents usually get wrong, so it is read from the dialogue instead of guessed:

| You… | The agent… |
|---|---|
| use the topic's terms, correct it on substance | skips the basics, doesn't repeat back what you said |
| ask what a term means, say "I don't get it" | explains the mechanism and stays at that level for the topic |
| give no signal | explains — an extra sentence is cheaper than an answer you can't follow |

**`/fill-me-in` for when an answer still lands badly.** The agent finds the invisible context its answer leaned on, re-checks key facts against the source instead of memory, and re-explains: the gist in one line, the mechanism, a step-by-step example, what follows. `/fill-me-in <part>` unpacks just that part.

## Plays well with "be brief"

Running [ponytail](https://github.com/DietrichGebert/ponytail), [caveman](https://github.com/JuliusBrussee/caveman) or your own "max three lines" rule? Keep them. Fill Me In tells the agent to let them cut code, repetition and pleasantries — never context, reasons or the mechanism — and repeats that with every prompt so it isn't drowned out. A direct request in your message ("code only") still wins over both. This priority is an instruction to the model, not a guarantee, and the eval has not measured it alongside those plugins yet.

## What it runs

Two `command` hooks, both `cat` of a text file inside the plugin: `hooks/rule.md` at session start and `hooks/reminder.md` on every prompt. Their output goes into the agent's context. No network calls, no scripts, no dependencies, nothing collected or stored. The `/fill-me-in` skill is a plain `SKILL.md`.

## Install

### Claude Code

```
/plugin marketplace add quiqqe/fill-me-in
/plugin install fill-me-in@quiqqe
```

Starts working from the next session. Update to the latest version from a terminal:

```
claude plugin update fill-me-in@quiqqe
```

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
