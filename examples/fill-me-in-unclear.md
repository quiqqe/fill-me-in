# "I don't get it" after a terse answer

Taken verbatim from the eval run in [`evals/results/latest.json`](../evals/results/latest.json), case `fill-me-in-unclear`.
Same model, same prompt; the only difference is whether Fill Me In is installed.

## Prompt

```
Earlier in this conversation you told the user:

"Fixed: dropped `set $php_fpm`, moved to upstream with zone + resolve, valid=1s. Recreating php no longer needs a reload."

The user replies: "I don't get it. What does that mean?"

Write your reply to the user.
```

## Without Fill Me In

> Sorry, that was too terse. Here's what was going on and what I changed.
>
> **The problem:** Nginx sends PHP requests to your `php` container by name. Docker gives a container a new IP address every time it's recreated. Nginx looked up that IP once and kept using it. So after you recreated `php`, nginx kept sending traffic to the old, dead address, and you got errors (usually 502 Bad Gateway) until you reloaded or restarted nginx.
>
> **The old workaround:** The config put the address in a variable (`set $php_fpm ...`). That's a common trick to make nginx look the name up again. It works, but it's fiddly, and it gives up some features, like connection reuse.
>
> **What I changed:**
> - **Removed the variable.** The PHP address is now defined in an `upstream` block, which is nginx's standard way to name a backend.
> - **Added `zone` and `resolve`.** Together these tell nginx to keep checking what IP the name `php` points to while it runs, not just at startup.
> - **Set `valid=1s`.** Nginx now re-checks the IP every second, so it notices a new container almost immediately.
>
> **What that means for you:** You can recreate or redeploy the `php` container and nginx will find the new one by itself. You no longer need to reload nginx afterward. For a second or so during the switch, a request might still fail.
>
> One caveat: `resolve` in an `upstream` block needs nginx 1.27.3 or newer. On an older nginx image, the config will fail to load. If you're not sure which version you're running, I can check.

## With Fill Me In

> I made a change so nginx finds the PHP container by itself after that container is recreated, so you no longer have to reload nginx by hand.
>
> One caveat first: I can't see your nginx config files from here (a search of the working directory found nothing). So this explanation comes from what I wrote earlier plus how nginx generally works, not from re-reading your files.
>
> **The problem.** nginx passes PHP requests to the PHP-FPM container by its name, something like `php:9000`. nginx doesn't use the name directly. It asks Docker's built-in DNS (at `127.0.0.11`) for the container's IP address. When you recreate the `php` container (after a rebuild or `docker compose up`), it can come back with a different IP. If nginx is still holding the old IP, it sends requests to an address that no longer exists and you get 502 errors until you reload nginx.
>
> **What each part of my earlier message means:**
>
> - **"Dropped `set $php_fpm`"**: the old config stored the address in a variable, e.g. `set $php_fpm php:9000; fastcgi_pass $php_fpm;`. That's a common trick to make nginx look the name up again on each request. I removed it.
> - **"Moved to upstream"**: an `upstream` block is nginx's standard way to name a backend. The PHP address is now defined there once, and `fastcgi_pass` points to that block.
> - **`resolve`**: a parameter on the server line inside the upstream. It tells nginx to keep re-checking DNS for that name in the background, instead of looking it up once at startup and never again. Free nginx has supported this since version 1.27.3.
> - **`zone`**: a small piece of shared memory where nginx keeps the latest address. `resolve` needs it.
> - **`valid=1s`**: set on the `resolver` line. It means nginx trusts a DNS answer for one second before asking again.
>
> ```
> request → nginx → upstream "php" → asks Docker DNS "where is php?" (every 1s) → php container
> ```
>
> **Example.** You rebuild the PHP container, and its IP changes from `172.18.0.5` to `172.18.0.7`. Within about a second, nginx's next DNS check returns `.7` and requests go there. Before the change, you'd have had to run `nginx -s reload` (or restart the nginx container) to stop the 502s.
>
> **What you need to do:** nothing. Just recreate the PHP container as usual. If you ever see 502s right after a PHP restart, check your nginx version with `nginx -v`. On anything older than 1.27.3, `resolve` won't work.
>
> I can't tell you why I preferred the `upstream` block over the variable, since both make nginx look the name up again. I no longer have that part of our conversation. The usual reason is that an `upstream` block is cleaner and can later hold several PHP containers, sharing requests between them.
>
> Does any part still need more explanation?
