# bw-site — Bullwinkle's Family Fun Center

A modernized, self-contained clone of [bullwinkles.com](https://bullwinkles.com),
served as a **pure static site** on the lab980 droplet (nginx + certbot, no
build step, no app server, no database).

## Layout

```
site/                     # the web root — everything nginx serves
  index.html              #   landing: hero video, Choose Location, Eat.Play.Repeat
  wilsonville/            #   Wilsonville, OR location page
  tukwila/                #   Tukwila, WA location page
  upland/                 #   Upland, CA location page
  assets/{css,js,img,video}
bin/bw                    # operate CLI (setup / deploy / vhost / rollback)
deploy/nginx.conf.template# static vhost (root current/, TLS via certbot)
DEPLOY.md                 # droplet runbook
```

`releases/` and the `current` symlink are created on the droplet by `bw` and
are git-ignored.

## The site

Faithful to the live site's branding (Francois One + Roboto, brand orange
`#EF5113` on deep navy) and content, using the real Bullwinkle's assets under
`site/assets/`. Each location page is a single scrolling page whose sticky nav
(Hours · Bowl · Play · Eat · Party · Events · Specials · Moose Perks · About)
anchors to on-page sections: hero, attractions, events & specials, birthday,
group events, Moose Perks, park map, FAQ, guest reviews, other locations, and a
full footer.

Accessibility: skip links, focus-visible rings, focus-trapped location modal,
keyboard-operable FAQ/carousel, and full `prefers-reduced-motion` support (the
carousel never auto-advances and the hero video is suppressed).

## Run locally

Any static file server from the web root:

```bash
cd site
python3 -m http.server 8080   # http://localhost:8080
```

No build step, framework, or dependencies.

## Deploy

See [`DEPLOY.md`](./DEPLOY.md). Short version, as root on the droplet:

```bash
provision-site bw ivjames/bw-site
ln -sf /var/www/bw/bin/bw /usr/local/bin/bw
bw setup        # -> https://bw.lab980.com
```

Routine updates: `bw deploy`.
