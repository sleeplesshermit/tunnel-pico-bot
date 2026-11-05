# tunnel-pico-bot

Slim tunnel-only stack for small panel / source-build hosts: one
**VMess-WS** inbound (xray) reachable only through a **Cloudflare tunnel**,
supervised by a ~150-line POSIX shell script. Zero npm dependencies.

Successor to `agsbx-nano-bot` / `agsbx-min-bot`, minus the general-purpose
installer: no extra protocols (Reality/Hy2/TUIC/AnyTLS/SOCKS5), no geo
routing files, no agents, no in-app keep-alive pings. On tunnel-only hosts
none of that was reachable anyway.

## Layout

| File | Role |
| --- | --- |
| `pico.sh` | The whole stack: fetch pinned binaries → render config + subscription → supervise |
| `index.js` | Node flavor supervisor (placeholder HTTP on `$PORT` + runs `pico.sh`); use where the platform starts `node index.js` |
| `start.sh` | Shell entry: `exec sh pico.sh` |
| `package.json` | `npm start` → `node index.js` |

Where a shell script can be the entry, prefer `pico.sh` alone — the Node
flavor costs ~30 MB of V8 RSS for the supervisor.

## Env

| Var | Required | Default | Meaning |
| --- | --- | --- | --- |
| `UUID` | yes | — | VMess UUID |
| `AGN` | yes | — | Tunnel hostname (Cloudflare ingress → `http://localhost:$VMPT`) |
| `AGK` | for tunnel | — | Cloudflare tunnel token. Empty = local test mode, no tunnel process |
| `VMPT` | no | `8001` | Local port of the VMess-WS inbound (127.0.0.1 only) |
| `PORT` | no | `3000` | Placeholder HTTP port (Node flavor only) |
| `SUB_PATH` | no | `sub` | Reserved for hosts that serve the subscription by path |
| `PICO_HOME` | no | `~/.tunnel-pico` | Binaries, config, `sub.txt`, `status.txt` |

State lands in `$PICO_HOME`: `bin/` (cached binaries), `config.json`,
`sub.txt` (base64 of the `vmess://` link — same format the earlier bots
produced), `status.txt` (core versions + start line, handy for weekly
checks). A copy of `sub.txt` is also written to the working directory.

## Pinned cores

- xray `v26.10.31` (official XTLS release zips, sha256-verified)
- cloudflared `2026.10.7` (official Cloudflare release binaries, sha256-verified)

Bump deliberately in `pico.sh`; hashes live next to the versions there.

## Run

```sh
UUID=... AGN=... AGK=... sh pico.sh      # shell flavor
npm start                                # node flavor (same env)
```

Clients import `sub.txt`. The tunnel ingress on the Cloudflare side must
point `AGN` at `http://localhost:$VMPT` (same setup as the earlier bots).
