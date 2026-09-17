# Packet Terminal

Live site: **https://packet-term.m0lte.uk**

A self-contained browser terminal that uses [`@packet-net/ax25`](https://www.npmjs.com/package/@packet-net/ax25) and [`xterm.js`](https://xtermjs.org) to drive a USB KISS modem from the browser. Late-80s phosphor-CRT aesthetic, TNC2 command set.

What makes this particular terminal interesting: the AX.25 state machine it walks comes from [`ax25sdl`](https://github.com/m0lte/ax25sdl), which is itself generated from the SDL diagrams in the AX.25 v2.2 specification. Updates to the spec's state-machine flow through into running code with no code changes here.

<img width="1065" height="746" src="https://github.com/user-attachments/assets/1d934e8f-a4d3-4695-95a6-930364fcd13d" />

Part of [pdn-web](../README.md), which holds one directory per browser app.

## Run it

Open `index.html` in **Chromium** or **Edge**. No build step. The page imports `@packet-net/ax25` and `xterm.js` from [esm.sh](https://esm.sh) at load time.

Web Serial isn't supported in Firefox or Safari. The page itself still loads (so you can read the help and see the layout), but `ATTACH MODEM` shows a "no Web Serial" notice.

## TNC2 commands

| Command | Meaning |
| --- | --- |
| `HELP`, `?` | List commands |
| `MYCALL <call>` | Set or show your callsign (e.g. `MYCALL M0LTE-1`) |
| `CONNECT <call>`, `C` | Open a link to a remote station |
| `DISCONNECT`, `D`, `BYE` | Drop the link |
| `STATUS`, `ST` | Show link state, mode, frame counters |
| `ECHO ON\|OFF` | Local-echo toggle (default ON) |
| `CLEAR`, `CLS` | Wipe the screen |
| `VERSION`, `V` | Runtime stack identity |
| `MON ON\|OFF`, `M` | Toggle monitor mode |

While connected, every keystroke is buffered and sent when you press Enter. `Ctrl-C` returns to command mode without disconnecting; type `D` to drop the link cleanly.

## Architecture

```text
                    ┌─────────────────────────────────┐
                    │           index.html            │
                    │                                 │
   keystrokes ──►   │   xterm.js  ──►  command parser │
                    │                  │              │
                    │                  ▼              │
                    │            @packet-net/ax25     │
                    │                  │              │
   USB Serial ◄───  │       WebSerialKissTransport    │
                    │                                 │
                    └─────────────────────────────────┘
```

`Ax25Stack` is constructed with a `WebSerialKissTransport` wrapping the `SerialPort` returned by `navigator.serial.requestPort()`. The TNC2 command parser maps text input to `stack.connect({ from, to })` and `session.write(bytes) / session.disconnect()` calls. Inbound bytes from `session.onData(...)` go straight to xterm.

## Scope

Connected-mode only - no UI/UNPROTO send, no `via` digipeater paths, no mod-128. `MON ON` opts in to BPQ-style passive frame tracing. The underlying library's full scope (what's in, what's out) is documented in the [`@packet-net/ax25` README](https://github.com/m0lte/ax25-ts#scope--whats-in-whats-out).

## How the live site updates

Pushes to `main` that touch `index.html` trigger [`.github/workflows/publish.yml`](.github/workflows/publish.yml), which uploads the file to OARC object storage. The OARC platform serves it back as the static site at `packet-term.m0lte.uk`. Round-trip from merge to live: ~10 seconds.

## Provenance

Extracted from `m0lte/packet.net` on 2026-05-17 (history preserved via `git filter-repo`) - the demo used to live at `web/ax25/examples/packet-terminal/` four levels deep in that monorepo. Now standalone: no build step, no `package.json`, no `node_modules`, just `index.html` and a tiny publish workflow.

## Sibling repos

| Repo | What it is |
| --- | --- |
| **`m0lte/packet-term-web`** *(here)* | Browser TNC2 emulator |
| [`m0lte/packet-term-tui`](https://github.com/m0lte/packet-term-tui) | C# Terminal.Gui TUI - same idea on the desktop |
| [`m0lte/ax25-ts`](https://github.com/m0lte/ax25-ts) | `@packet-net/ax25` - the library this demo consumes |
| [`m0lte/ax25sdl`](https://github.com/m0lte/ax25sdl) | SDL transcriptions + codegen - transitively consumed via `@packet-net/ax25` -> `ax25sdl` |
| [`m0lte/packet.net`](https://github.com/m0lte/packet.net) | .NET libraries + node host (the C# side of the stack) |

## License

[MIT](LICENSE). Proudly built with AI assistance.
