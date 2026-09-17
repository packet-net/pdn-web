# pdn-web

Packet radio in a browser tab. One directory per app, each self-contained, each published the way
it already was.

| | |
| --- | --- |
| [`packet-term-web/`](packet-term-web/) | A TNC2 terminal for a KISS modem on a serial port. `@packet-net/ax25` + xterm.js + Web Serial, late-80s phosphor CRT. Live at **https://packet-term.m0lte.uk** |
| [`soundmodem-web/`](soundmodem-web/) | A station on a sound card: every NinoTNC mode, connected-mode sessions with a keyboard, levels with a meter, and a transmitter test. `@packet-net/soundmodem` + `@packet-net/ax25`. Live at **https://packet-net.github.io/pdn-web/soundmodem-web/** |

They are deliberately not combined. They are different apps for different radios, and the day one
of them needs a build step or a framework it can have one without dragging the others along.

## How each is published

Two pipelines, because the two apps already had two and there was no reason to make them one.

- **packet-term-web** is uploaded to OARC object storage on every push that touches it, and served
  from its own domain. `.github/workflows/publish-packet-term-web.yml`.
- **soundmodem-web** goes to this repository's GitHub Pages site, one subdirectory per app under
  `packet-net.github.io/pdn-web/`. `.github/workflows/pages.yml`.

A new app picks whichever suits it and adds itself to the matching workflow.

## Versions

Neither app has a build step: both import their packages from a CDN at run time, so the version
they run is whatever the page asks for.

`soundmodem-web` asks for the newest `0.x` of `@packet-net/soundmodem` rather than naming a
release. That page deploys on a push and the package only on a release, so an exact pin meant
every release left the page behind until something re-pinned it. Note that a caret range will not
do this: on a `0.x` version caret means patch-only, and `@packet-net/soundmodem@^0.69.0` resolves
to 0.69.0. The bare major is what tracks minor releases. The cost is that a CDN caches a floating
range for hours, so a new release reaches the page a little after it reaches npm.

Where a page uses something the CDN is not serving yet, it disables that control and says which
version it wants rather than throwing.

## Licence

AGPL-3.0-or-later. `@packet-net/soundmodem` carries the GPL-3.0-or-later modem core inside an
AGPL-3.0-or-later combination, which is what GPLv3 section 13 provides for, and anything importing
it forms a combined work under those terms. Hosting one of these pages is conveying it: offer the
source.
