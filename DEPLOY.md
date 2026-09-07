# Deploying haunted.lab980.com

Haunted Hotel is a **fully static** site on the lab980 droplet: one
`index.html`, no build, no app process, no port, no pm2, no `data/`. Same
shape as sparkle-butt. `provision-site` doesn't fit (its vhost is
proxy-shaped), so this repo ships its own `bin/haunted` CLI that does the
whole bring-up with the standard doctl / security-headers / certbot shape.

## First-time bring-up (on the droplet, as root)

```sh
git clone https://github.com/ivjames/haunted-hotel /var/www/haunted
ln -sf /var/www/haunted/bin/haunted /usr/local/bin/haunted
haunted setup
```

`haunted setup` is idempotent and does, in order:

1. **DNS** — `A haunted.lab980.com -> droplet IP` via doctl (skipped if the
   record exists; `--no-dns` to skip entirely, `--ip` to override the
   auto-detected droplet IP).
2. **nginx** — a static vhost at `/etc/nginx/sites-available/haunted.lab980.com`
   rooted at the checkout, with the shared
   `snippets/lab980-security-headers.conf` include (seeded if this is a
   fresh box), symlinked into `sites-enabled/`, `nginx -t` + reload.
   Dotfile paths (`.git`, `.env`) are denied.
3. **TLS** — waits for DNS to resolve to the droplet (up to ~5 min), then
   `certbot --nginx -d haunted.lab980.com --redirect -n`. `--no-tls` to
   skip; the command to run later is printed if DNS isn't ready yet.

## Deploys

> **`haunted deploy` does not work in this repo as it stands.** It runs
> `git fetch origin main` + `git reset --hard origin/main`, and **this repo has
> no `main` branch.** Its only branches are `claude/haunted-hotel-game-wo0gy4`
> (the default, and what the live site is built from),
> `claude/lab980-conventions-sync-2026-09-04` and `claude/nginx-t-rollback`.
> `bin/haunted` runs under `set -euo pipefail`, so the `git fetch origin main`
> fails and the command aborts before the reset — it does not damage anything,
> but it never deploys either. Verified 2026-09-07:
> `git ls-remote origin main` returns nothing.
>
> Until `bin/haunted` grows a branch variable (every sibling CLI has one —
> `SHEEP_BRANCH`, `SPARKLE_BRANCH`, …; this one hardcodes `origin/main` at
> `bin/haunted:188-189`), deploy by hand with the branch this repo actually
> has:
>
> ```sh
> cd /var/www/haunted
> git fetch origin claude/haunted-hotel-game-wo0gy4
> git reset --hard origin/claude/haunted-hotel-game-wo0gy4
> ```

Nothing to build, nothing to restart: nginx serves the file straight from
the checkout.

## Verify what is actually live

A 200 only proves nginx answered, not which build it served. Compare the
served file against the commit you expect:

```sh
git fetch -q origin claude/haunted-hotel-game-wo0gy4
curl -s https://haunted.lab980.com/index.html | git hash-object --stdin
git rev-parse origin/claude/haunted-hotel-game-wo0gy4:index.html
```

Identical hashes mean the deploy landed. Fetch first and compare against the
`origin/` ref, not a local branch — a stale clone and a stale deploy hash
identically, so the local-branch form of this check passes in exactly the case
it exists to catch. Verified 2026-09-07: both sides read
`0d67204274d0e18e70d40a0a9be1b8e13e7a3f4f`, so the live site matches the
branch tip.

## Everything else

```sh
haunted status     # vhost enabled? cert present/expiry? git rev? public 200?
haunted remove     # tear down vhost + cert + DNS record (repo dir stays)
```

`health-check` (from the lab980 repo) covers this site as a static vhost:
DNS, public https, cert expiry. There is nothing for its pm2 or systemd
sections to see.

## Notes

- **`*.md` is not denied on this site** (as of 2026-09-07), unlike its sibling
  static sites and unlike the assertion in
  `.claude/rules/lab980-conventions.md` ("the vhost denies dotfiles and
  `*.md`"). Verified from outside: `/README.md` and `/DEPLOY.md` both return
  **200**; `/.git/config` returns 403, so the dotfile deny in step 2 above is
  real. The web root is the git checkout, so everything tracked here is
  public — don't commit anything you wouldn't publish. PR #3 in this repo adds
  the `*.md` deny to the vhost; this note describes the state before it lands.
- **This repo has no `main` branch**, and nothing here deploys from one. See
  the box under "Deploys". The lab980 conventions call this out too: the
  default branch is not always `main`, and this repo is the case with no
  `main` at all.
