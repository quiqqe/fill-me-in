# "I don't get it" after a terse answer

Taken verbatim from eval run 4 ([`evals/results/latest.json`](../evals/results/latest.json)), case `fill-me-in-unclear`.
Same model, same prompt; the only difference is whether Fill Me In is installed. The "without" reply is
one that failed the judge, the "with" reply one that passed.

## Prompt

```
Earlier in this conversation you told the user:

"Fixed: dropped `set $php_fpm`, moved to upstream with zone + resolve, valid=1s. Recreating php no longer needs a reload."

The user replies: "I don't get it. What does that mean?"

Write your reply to the user.
```

## Without Fill Me In

> Sorry, that was too much shorthand. Here's what it means in plain terms.
>
> **The problem:** nginx sends PHP requests to your `php` container by name. Every time that container is recreated (a rebuild, restart or `docker compose up`), Docker gives it a new IP address. nginx looked up the IP once at startup and kept using it. So after a recreate, nginx kept sending requests to an address where nothing was listening. You'd get 502 errors until you reloaded nginx.
>
> **What I changed:** I changed the nginx config so it looks up the `php` container's address again every second, instead of looking it up once and keeping it.
>
> - **"Dropped `set $php_fpm`"**: The old config used a variable to force nginx to look up the address again. It's a common workaround, but it's clunky and has side effects. I removed it.
> - **"Moved to upstream with zone + resolve"**: I replaced it with nginx's built-in way of doing this. An `upstream` block names the PHP backend. Adding `resolve` tells nginx to keep looking up that name. `zone` is a small piece of shared memory that `resolve` needs in order to work.
> - **"valid=1s"**: This tells nginx to trust a looked-up address for only one second before checking again.
>
> **What it means for you:** you can recreate the PHP container whenever you like, and nginx will pick up its new address within about a second. You no longer need to reload or restart nginx afterward.

## With Fill Me In

> **In short:** nginx used to lose track of your PHP container whenever that container was recreated. I changed the config so nginx re-checks the container's address on its own, so you no longer have to reload nginx after recreating PHP.
>
> **What was going wrong**
>
> nginx passes PHP requests to the PHP container (PHP-FPM) by name, something like `php:9000`. Docker turns that name into an IP address, for example `172.18.0.5`. By default nginx does that lookup once, at startup, and keeps using that IP.
>
> When you recreate the PHP container, it usually comes back with a new IP, say `172.18.0.7`. nginx keeps sending requests to the old `172.18.0.5`, where nothing is listening anymore. That's why you got 502 errors until you reloaded nginx, which made it look the name up again.
>
> **What I changed**
>
> 1. **Removed the `set $php_fpm ...` workaround.** That trick stores the address in a variable, which makes nginx look the name up on every request. It does work, but it's awkward, and it skips nginx's normal connection handling for backends.
> 2. **Defined PHP as an `upstream` block with `zone` and `resolve`.** An `upstream` is nginx's named list of backend servers. `resolve` tells nginx to keep re-checking what IP the name `php` points to while it runs. `zone` gives nginx a small piece of shared memory to store the updated address, and `resolve` needs it to work.
> 3. **Set `valid=1s` on the resolver.** This tells nginx to trust a lookup for only one second before asking Docker again. After the PHP container is recreated, nginx finds the new IP within about a second.
>
> **What it means for you**
>
> You can run something like `docker compose up -d --force-recreate php` and nginx keeps working without `nginx -s reload`. You might see an error or two in the second or so while the container restarts, but it fixes itself without you doing anything.
>
> One caveat: `resolve` inside an `upstream` needs nginx 1.27.3 or newer (older versions only had it in the paid nginx Plus). If your nginx image is older, nginx will refuse to start with this config. If that happens, tell me and I'll change it back to the variable approach.
