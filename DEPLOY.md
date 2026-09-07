# Deploy — Bullwinkle's site (lab980 droplet)

Matches the lab980 one-dir-per-site / nginx / certbot shape. This is a **pure
static site** — no build step, no app server, no database — so there's nothing
to migrate or keep running under pm2. `bw setup` publishes the static web root
behind nginx and issues TLS; `bw deploy` re-publishes on every update.

The one hard requirement carried over from the convention:

- **Atomic publishes.** Copy the web root into a fresh `releases/<ts>/` dir and
  swap the `current` symlink; never rsync into the served dir in place, or a
  visitor mid-deploy can load `index.html` against half-updated assets.
  `bw deploy` does this (and `bw rollback` swaps back).

Everything site-specific lives in the project's **`bin/bw` operate CLI** (the
lab980 per-site tooling convention), so bring-up is a single command.

## First-time provisioning

> **Don't export `GITHUB_TOKEN` for this.** `ivjames/bw-site` is a **public**
> repo — verified 2026-09-07, the GitHub API reports `private: false` — so the
> clone needs no credential. This section used to say otherwise and tell you to
> export a token into the shell that then runs `provision-site`.
>
> That is not a harmless surplus step on this droplet. pm2 copies the
> environment of whatever shell starts a process into the process *and* into
> `~/.pm2/dump.pm2`, where it acquires an indefinite on-disk lifetime; the
> 2026-09-05 audit found `GITHUB_TOKEN` in **18 of 21** pm2 registrations, none
> of which had asked for it. An exported token in the provisioning shell is one
> of the ways it got there — and here it bought nothing.


Run as **root on the droplet**. Subdomain `bw.lab980.com` throughout; change the
`bw` label if you want a different one (and set `BW_FQDN` for `bw setup`).

```bash
# 1. Subdomain shell: DNS + clone + dir. One command.
#    (ivjames/bw-site is PUBLIC — no token needed. See the note below.)
provision-site bw ivjames/bw-site

# 2. Symlink the operate CLI onto PATH (once), then let it do the rest:
#    first atomic publish -> static vhost -> TLS.
ln -sf /var/www/bw/bin/bw /usr/local/bin/bw
bw setup
```

That's it — `https://bw.lab980.com` is live. `bw setup` overwrites
`provision-site`'s default proxy vhost with the static vhost (root
`current/`), then issues the cert against it, so the vhost shape is handled for
you.

If certbot reports it can't reach the host, DNS from step 1 is still
propagating — just re-run `bw vhost` a minute later.

> Want a prettier hostname? `BW_FQDN=bullwinkles.lab980.com bw setup` (and point
> that DNS record at the droplet). The label passed to `provision-site` only
> decides the `/var/www/<label>` dir + default subdomain.

## Routine redeploys

```bash
bw deploy       # pull main -> publish releases/<ts> -> swap current -> reload nginx
```

Other operate commands:

- `bw publish`   — re-publish the current checkout without pulling (local edits)
- `bw rollback`  — swap `current` back to the previous release
- `bw releases`  — list retained releases (newest first)
- `bw vhost`     — rewrite the nginx vhost + re-issue TLS

## Verify after deploy

A 200 only proves nginx answered, not which release `current` points at. Ask
for the deployed commit as well as loading the page:

```bash
bw releases                       # newest first; `current` should be the top one
git -C /var/www/bw rev-parse HEAD
git -C /var/www/bw fetch -q origin main && git -C /var/www/bw rev-parse origin/main
```

The two SHAs should match after a deploy. Compare against `origin/main` after a
fetch rather than local `main`: a stale clone and a stale deploy hash
identically, so the local-branch form of the check passes in exactly the case it
exists to catch. Note that the blob-hash comparison used on the other lab980
static sites does not apply here — the served root is `current/`, a symlink into
`releases/`, not the checkout.

- `https://<fqdn>/` loads over TLS (padlock) and shows the location picker.
- The hero video autoplays; the *Eat. Play. Repeat.* carousel advances.
- `/wilsonville`, `/tukwila`, `/upland` each load with the correct city,
  phone, address, and cross-links.
- No mixed-content or 404s in DevTools → Network (all assets are same-origin
  except the Google Fonts stylesheet).
