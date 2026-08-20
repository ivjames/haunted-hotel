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

```sh
haunted deploy     # git fetch + reset --hard origin/main — that's the whole deploy
```

Nothing to build, nothing to restart: nginx serves the file straight from
the checkout.

## Everything else

```sh
haunted status     # vhost enabled? cert present/expiry? git rev? public 200?
haunted remove     # tear down vhost + cert + DNS record (repo dir stays)
```

`health-check` (from the lab980 repo) covers this site as a static vhost:
DNS, public https, cert expiry. There is nothing for its pm2 or systemd
sections to see.
