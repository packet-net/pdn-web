# CLAUDE.md

Operating notes for Claude Code (and other agents) working in `packet-net/pdn-web`.

## What this repo is

Browser apps for packet radio, one directory each, each self-contained. See
[README.md](README.md) for what they are and where each one is served from.

They are deliberately not combined, and shared code between them is not a goal in itself: they
are different apps for different radios, and a helper worth sharing is usually a helper worth
publishing as a package instead.

## How to work in here

No build step in any app. Edit the HTML, serve the directory over HTTP, reload in Chromium. Web
Serial, WebHID and the microphone all need a secure context, so `localhost` qualifies and
`file://` does not.

```sh
python3 -m http.server 8000     # from the app's directory
```

## Hard rules

### Don't bundle, don't add build tooling

The point of every app here is "an HTML file, no build, drop it on any static host". If you find
yourself reaching for vite, webpack or rollup, stop and ask. The ESM-via-CDN approach is
deliberate: it lets somebody fork the file, change a colour and host it themselves.

`soundmodem-web` has a `package.json` and a `node_modules`, and they are for its page probe and
for `?local` development only. The page itself still imports from a CDN and still has no build.

### The deploys are publicly visible

Every app here is a public site. What you push affects whoever is using it right now. Test
locally, and run the app's own tests where it has them.

### Don't commit secrets

Deploy credentials and API tokens stay outside the repo. The `oarc-static-upload` skill keeps
OARC's in `~/.config/oarc/`.

### Runtime logic belongs upstream

Anything that would belong in `@packet-net/ax25` or `@packet-net/soundmodem` belongs in those
repositories. What lives here is UI, command parsing and the glue between the two. If a change
feels like "the library should expose X so the app can do Y", file it against the library.

## Per app

### packet-term-web

A TNC2 terminal over Web Serial KISS. Extracted from `m0lte/packet.net` on 2026-05-17
(originally `web/ax25/examples/packet-terminal/`) so it could have its own deploy cadence and
issue tracker, and moved into this repository on 2026-09-17.

- **Pin npm versions exactly in the import URL**, never `@latest`. A pin means the deployed page
  does not break silently when an upstream publishes a bad release. Bump in one PR and check it
  in a browser before merging.
- **Web Serial is Chromium only.** Do not feature-detect and polyfill Firefox or Safari. They do
  not support it, and the only honest UI is the existing "no Web Serial" modal.
- Deployed to OARC object storage by `publish-packet-term-web.yml`, served from
  packet-term.m0lte.uk.

### soundmodem-web

A station on a sound card. Moved here from `pdn-soundmodem/web/demo` on 2026-09-17, where it had
outgrown being a demo inside the modem's own repository: it deploys on a push while the modem
deploys on a release, and it was running the modem's whole .NET suite to publish an HTML file.

- **It floats the modem's version rather than pinning one**, which is the deliberate exception to
  the rule above and not the same thing as `@latest`: it asks for the newest `0.x`, which is
  bounded, and it feature-detects every control it offers so an older package disables the
  control and says which version it wants rather than throwing. A caret range would not work
  here, because on a `0.x` version caret means patch-only and `@^0.69.0` resolves to 0.69.0.
- **Run the page probe before pushing.** `npm test` in that directory runs the page's own script
  in Node against the real package. It has caught every UI defect this app has had.
- Deployed to this repository's GitHub Pages by `pages.yml`.
