---
name: fill-me-in
description: Re-explain my previous answer to someone who did not follow it because they did not watch the work happen or do not know the domain. Use on /fill-me-in, or when the user complains that an answer already given is unclear — "I don't get it", "what do you mean", "what's the point", "fill me in", «не понял», «непонятно», «в чём суть», «что ты имел в виду». Do not use for a plain request to explain a new topic ("explain how X works") — just answer that. Optional argument: the specific part or concept to explain.
---

# Fill Me In

The previous answer did not land: the words are familiar, the point is not. The cause is almost
always the same — the answer was written from inside the process and relied on context only I saw.
The task is to explain so that a smart person who did not see the process and does not know this
domain gets the point. Answer in the user's language.

## What to explain

- If the user said what exactly is unclear (as the argument or in their own words), that is the main
  guide. Their words are more reliable than my judgment: the answer being unclear already shows
  I misjudge what is obvious.
- Otherwise: my last substantive answer as a whole.

## Steps

1. **Find the missing context.** List what the answer leaned on that the reader could not see:
   events from the process ("after the second run"), my internal names, dropped hypotheses,
   mechanisms I took as obvious, unstated reasons.
2. **Re-check facts against the source.** Verify key claims in the code, config, log or command
   output, not from memory — otherwise the earlier mistake repeats in a confident tone. If you
   cannot verify something, say it was an assumption. If the process details are gone from context
   (the conversation was compacted), say "I can no longer see that" instead of reconstructing it
   by guesswork.
3. **Open with the gist in one sentence.** What happened or what I am claiming, in plain words,
   so the reader knows where the explanation is heading.
4. **Describe the situation.** One or two sentences: what the task or problem was and what I was
   trying to find out.
5. **Explain the mechanism.** If the conclusion depends on how something works (Docker networking,
   DNS caching, a queue, a transaction), explain how it works — only the part needed for the
   conclusion, in your own words. Define an unavoidable term on first use. For a chain of steps
   or components, draw a text diagram with arrows.
6. **Show an example.** A concrete scenario, step by step, with real names from the project:
   here is the request, here is what happens, here is the result and why.
7. **Connect it to the conclusion.** What follows, what it risks or gives, what the user should do
   (or "nothing, this is just an explanation").
8. **End with one line** asking whether anything is still unclear.

## Example

An unclear answer (written from inside the process):

> Dropped `set $backend`, moved to an upstream with `zone` and `resolve`, `valid=1s`. Recreating
> app no longer needs a reload.

The same answer after unpacking:

> **Gist:** nginx now notices on its own that the app container was recreated, so it no longer
> needs a restart.
>
> **How it works.** nginx forwards requests to the app container by the name `app`, and Docker's
> network DNS (`127.0.0.11`) turns that name into an IP address. When the container is recreated it
> can get a new IP. If nginx remembered the old one, it sends requests into the void. Previously
> a variable saved us: with `set $backend app:8080`, nginx asks DNS again at request time. Now
> the address lives in an `upstream` block with the `resolve` parameter: nginx (open-source, since
> 1.27.3) re-queries DNS by itself; `valid=1s` sets how often — once a second; `zone` reserves
> memory where the fresh address is kept.
>
> ```
> request → nginx → upstream backend → DNS 127.0.0.11: "app = ?" (every 1 s) → app container
> ```
>
> **Example.** A deploy recreates app and its IP changes from .5 to .7. Within a second nginx learns
> the new address and sends requests to .7. Before, nginx would have had to be restarted.
>
> **Why it matters.** Besides convenience, an `upstream` can later hold several app instances:
> nginx will spread requests between them and retry on another one if one fails.
>
> Anything still unclear?

## Don't

- Don't repeat the previous answer in other words: without the missing context it is not an explanation.
- Don't lecture on the whole topic. Explain exactly what the conclusion cannot be understood without.
- Don't compress for the sake of plugins like ponytail or caveman: clarity matters more here.
- Don't present a guess as a fact.

## Check before sending

Could someone who reads only this answer retell the point to a colleague in their own words?
If not, find the missing step and add it.
