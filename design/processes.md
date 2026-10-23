# Processes and crates

The rule that lays out the code: **a process holds exactly the authority it acts on**, and a
crate answers the question "on whose authority does this act?"

## Processes

| Process | What it is | Talks to | Over |
| --- | --- | --- | --- |
| `kdeviced` | the device daemon; when it runs, starts one `kuserd` per registered user, and reads one entry of the platform keychain, the device operational key, for a supervised child that asks; it can reach no user's data | spawns `kuserd`; the operator CLI `kctl` | a device socket with challenge-response |
| `kuserd` | the user daemon; opens the platform keychain and reads from it the device operational key and the roots of every user it serves; holds the keys, store, apps, windows and sync engine of those users, and the one peer listener: each user on the machine when `kotton-start` runs it, one user when `kdeviced` starts it | the CLI and REPL, the shell, the sidecars, peers | the user socket; QUIC with mutual TLS to peers |
| `kuid` | the compositor: owns the draw surface, composites app windows, routes input, draws prompts | `kuserd` only | a socket pair handed to it at spawn, so there is no path to impersonate it on |
| `kinferd` | the inference runtime: no key, no store, no network | `kuserd` only | a pair of pipes at spawn; kernel sandboxing on Linux |
| `ksyncd` | watches the local tree and triggers sync; no key | `kuserd` | the user socket with a bootstrap token |
| `kquicd` | accepts incoming QUIC connections and hands them to the daemon after checking their trust floor; no key | remote devices; `kuserd` | QUIC in; the user socket out |
| `khttpd` | the HTTP and WebSocket gateway for a browser; holds a remote-origin token and can reach no local-tier verb | a browser; `kuserd` | HTTPS in; the user socket or a relayed circuit out |
| `kvaultd` | the vault: keeps sealed packs it cannot open; no apps, no display, no plaintext verb | devices, the relay, the landlord | QUIC with seven verbs; a local control socket; HTTP to a shelf |
| `kotton-relay` | address exchange and circuits; no inbox | nodes, vaults, gateways | HTTP and two WebSocket routes |
| `kotton`, `kctl` | the CLI and REPL for a user; the device-owner CLI | `kuserd`; `kdeviced` | the sockets, with a session token |

What runs depends on what starts it. The start script an install carries, `kotton-start`, and
`just desktop::run` run one `kuserd` and no `kdeviced`, and that one daemon loads every user
registered on the machine, so the keys of every unlocked user are in one process. Inside it,
users are kept apart by the session (each token names one user, and a request acts for the user
its token names), by each user's store under `users/<id>/` with a database of its own, and by
each user's keys. Started by `kdeviced`, a `kuserd` is told one user: it loads that user and no
other, refuses to create another, and listens on a socket named for that user, which `kotton`
reaches only when `KUSERD_IPC_NAME` names it and `kctl` dials by name. Either way, every
`kuserd` on a machine opens the two machine databases, reads the one peer port, and opens the
platform keychain scoped to the install's base directory. `kdeviced` answers a supervised child
authorized at its spawn with the device operational key, and it registers none: the `kuserd`
it starts are not supervised children, so they read their roots from the keychain themselves.
Outside the daemons, `kctl` keeps its operator key in the keychain, `kotton device reset`
deletes the install's entries, and `kvaultd` keeps its own key in the store of its state
directory. One process per user on the path an install runs, and one process reading the
keychain for every user on the machine, are on the [not-built list](not-yet.md).

Every request on every socket has a trust tier, assigned in one exhaustive match: a new
request is a compile error until someone assigns it one. A remote caller reaches a gateway
that holds no key and no state, and from there only the verbs its tier allows.

The REPL path builds and runs with no compositor, no guest app and no GTK (`just
headless::build`). The window adds `kuid`, the GPU renderer and the three bundled apps; the
daemon embeds the apps only when they exist, and their absence is an ordinary state.

## Crates

`core/` is privileged code, laid out by whose authority each crate acts on.

- **Mechanism, nobody's authority.** `common` (conventions and constants), `sys` (the
  platform abstraction), `pair` (the pairing handshake, with no I/O of its own), `proc`,
  `time`, `infer` (the inference mechanism).
- **Carriers.** `context` holds per-user and per-device data and handles, never behaviour.
- **Primitives.** `wire` (every wire format and persisted shape; the request and response
  enums live here and their surfaces are pinned to committed files, and a field that carries a
  token, a code, a credential or a key prints as `<redacted>` and crosses the wire unchanged),
  `schemas`, `crypto`, `net`, `fs`, `diag`, `ipc`.
- **Domains, one principal each.** `identity`, `peer`, `record`, `sealed` (address a store's
  objects without the means to open them), `store`, `sync`, `org` (an organisation that
  splits its key by threshold signing), `grant`, `msg`, `app`, `outbound`, `surface`,
  `secret`.
- **Two authorities in one crate.** `device`: its `runtime` acts on the machine's (the
  device's operational key and the one peer listener), its `users` module on that of each
  user the process serves (it resolves their roots and opens their stores). `kuserd` builds
  it for its users and `kvaultd` for its pools, through the same module. Separating the two
  is on the [not-built list](not-yet.md).
- **Enforcement, which decides who is asking.** `session`, and the daemons above.
- **The sandbox.** `wasm`, which loads a component, satisfies its imports, and runs the grant
  check on every call.

`client/` has no authority of its own and asks: `kotton-client` (the SDK the CLI uses),
`shell` (the command engine, compiled natively for the REPL and as a guest for the window),
`cli`, `web-pair` (the browser's half of pairing), `record-file`.

`guest/` is confined code and is not in the workspace: `sdk/kotton-app` (the contract and
SDK), `apps/` (the project's own apps), `examples/` and `extensions/`.

`infra/relay` is the relay. `infra/release` is the installers and the start script. `tests/`
is the integration suite, which runs real daemons, a real vault and a real relay on one
machine. `xtask/` is the build and check tooling.

A dependency rule between the regions is checked at build: `core` may not depend on `client`
or `guest`, and `client` may not depend on `guest`.

## Where to look first

- `core/wire/src/ipc/` for everything the daemon can be asked.
- `core/kuserd/src/agent/` for what it does about each request.
- `core/store` and `core/sealed` for how a thing is kept.
- `core/msg` for conversations and sharing.
- `core/kvaultd` and `infra/relay` for the rented instances.
- `client/shell/src/commands/` for every verb a person types.
- `tests/tests/suite/` for how the whole thing is exercised end to end.
