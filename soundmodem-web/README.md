# soundmodem-web

A packet station in a browser tab, with a sound card where the TNC goes.

Live at **https://packet-net.github.io/pdn-web/soundmodem-web/**

The DSP is [`@packet-net/soundmodem`](https://www.npmjs.com/package/@packet-net/soundmodem), the
pdn-soundmodem core compiled to WebAssembly, and the link layer is
[`@packet-net/ax25`](https://www.npmjs.com/package/@packet-net/ax25). Both load from a CDN at run
time, so this directory is one HTML file and there is no build step.

## What it does

- Every mode the modem catalogue builds, including the NinoTNC set.
- Connected mode: C and D, and a line of text you type into the session and press enter. It works
  from either end, so a station connecting to you lands in the same controls with nothing clicked.
- A monitor half and a session half, with a handle between them you can drag.
- RX and TX levels in dB with a meter, a TXDELAY slider, and a transmitter test: two tones for a
  linearity check, or one tone for a carrier level or an FM deviation check by Bessel null.
- PTT on a serial RTS/DTR line or a CM108-family dongle's GPIO over WebHID.
- The station is remembered in the browser it was set up in, including which keying interface, so
  a reload does not mean setting it up again.

## Run it

Chrome, Edge or Opera on desktop, over https or localhost: the microphone and the keying device
both need a secure context, and neither Web Serial nor WebHID exists in Firefox or Safari.

```sh
python3 -m http.server 8080      # from THIS directory
```

Then open `http://localhost:8080/`. Add `?local` to load the modem from the copy installed here
rather than from the CDN, which is what the page probe uses and what you want when developing the
package and this page together:

```sh
npm install                                    # the pinned version
npm link @packet-net/soundmodem                # or a pdn-soundmodem working tree
```

For the second, run pdn-soundmodem's `web/build.sh` first so the WebAssembly bundle is there.

## Testing it

```sh
npm install && npm test
```

The page probe runs this page's own script in Node, against the real package, with a shimmed DOM
and a fake Web Audio graph. It loads the page twice against one browser store - once driving every
control and a whole session, once to prove the station comes back - and it runs a second time
against a package with the newest methods stripped off, to prove the page disables what it cannot
drive rather than throwing.

It exists because everything else here is somebody else's library and that leaves a gap the size
of the whole browser. A mistyped element id, a handler on the wrong event, a slider that moves and
reaches nothing: none of it is visible to a decode test, and all of it is ordinary JavaScript that
Node runs exactly as a browser does.
