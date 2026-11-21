# 0001: The product is repositories along process boundaries, joined by the protocol

## Context

Kotton exists so that the people who generate data are the people who own it. Ownership is
value and standing: who holds the thing, who can be paid for it, and who must answer when it
is demanded. A person who owns their data can be asked for it, and nobody else can be asked
instead, because nobody else has it to give.

The system that makes that true is several processes, each holding exactly the authority it
acts on, talking over channels. A daemon holds a person's keys and store. A compositor owns
the display and draws the prompts an app must never draw. A sandbox host runs apps and holds
no keys. A vault keeps sealed packs it cannot open. A relay exchanges addresses and carries
circuits and holds no inbox. A gateway is the door the network knocks on and can reach no
local-tier verb. Each is a trust level, and the boundaries between them are where the design
lives.

A boundary between processes is a contract: what one may ask of another, what crosses with the
request, what each side must verify, and what the answer means. That contract is the protocol,
and it is what makes the pieces composable: anyone may write a client, a compositor, a desktop,
a vault or a gateway, and a person may install the one they trust.

What, then, is a repository: which boundaries are drawn as repositories, what each holds, and
what joins them?

## Decision

The product is several repositories, one per process boundary that holds its own authority,
joined by the protocol: this repository, which names every request and reply one process can
make of another, what crosses with it, and what it means; versioned; depended on by number.

| Repository | What it is |
|---|---|
| `kotton-protocol` | the contract: the protocol and type crates; what crosses and any side must verify (the pack envelope, log verification, the admission handshake, the pairing types, the native-guest contract); the client library; the app surface, defined once, with WIT as its Wasm binding; the handle type that is a file descriptor on today's platforms and a capability later; the RFCs; the pages that describe the whole; the versioning rule |
| `kotton` | the system: the user daemon, the device daemon, the QUIC acceptor, the sync watcher, the device-owner tool, and the sandbox host as a process of its own that holds no keys; ships the system archive |
| `kotton-cli` | the CLI and its REPL mode: unprivileged, built from the protocol and the client library alone and held to that by a check; ships its archive |
| `kotton-relay` | the relay: address exchange and circuits between devices, vaults and gateways, and one open carried to another relay for a device that stands at it (RFC 0007); holds no inbox and reads no pack; ships its archive |
| `kotton-vault` | the vault: keeps sealed packs it cannot open, for devices, the relay and the person who runs it; no apps, no display, no plaintext verb; ships its archive |
| `kotton-gateway` | the door from the network: `khttpd`, the HTTP and WebSocket process a browser reaches; trust path, holding a remote-origin token and reaching no local-tier verb; one implementation among any that speak the gateway's surface of the protocol |
| `kotton-portal` | the page a person opens from any browser, on any device, to reach their things at home, and the pairing page: an unprivileged client, built from the protocol and the client library alone and held to that by a check; one client among any |
| `kotton-compositor` | the reference compositor: the trusted display path, the prompts an app must never draw, input routing; one compositor among any that speak the compositor surface |
| `kotton-desktop` | the reference desktop: the home screen, the dock, the overlays, the first-party apps; an app like any other, one desktop among any |
| `kotton-sdk` | the language bindings of the app surface |
| `kotton-infer` | the inference runtime: the first native guest, confined by the system's sandbox host under the native-guest contract |
| `kotton-dist` | the distribution: which version of each archive goes together, the installers, the launcher, the nine lines a person types to install and run it, the signed sums over the set |
| `kotton-ahead` | the reference tree: the whole in one tree, read-only, from which a repository is cut when no other tree holds its code |
| `kotton-kernel` | reserved: the capability kernel, the last implementation of this protocol; empty until the protocol is stable enough that bare metal is a defined task |

No repository but this one defines a request. A change a dependent needs lands here first, as
a new version; the dependent takes the version. Each repository tags and ships its own
archive; only the distribution says what goes together; a person installs a distribution.

A repository boundary is drawn where there is a versioned contract between the sides. A
crate boundary is drawn where there is an authority boundary. The system's daemons share
crates that are the system's insides, not the contract, and stay in one workspace, each
binary held to the crates it may link.

A repository is cut from the tree that holds its code. `kotton` holds the code of the
contract's crates, the client library, the CLI, the relay and the vault, with the system's, in
which the sandbox host is not yet a process of its own; `kotton-dist` holds the distribution's;
the reference tree, `kotton-ahead`, holds the rest. Until a part is cut, `kotton`'s tree is laid
out as the table above: a top-level directory for each part that is cut, and the system's
directories, each named in the table's row it belongs to, and a check in `kotton`'s gate that
holds each directory to what that part may link. An edge the table does not allow is on that
directory's owed list, by name, or the check fails. Each list is printed on every run, and the
check refuses a list that grew, but for one that grew because an RFC narrowed its row.

A cut is a move, with its history. What a part may link is settled in the tree that holds it,
before its cut, and a part is cut when its owed list is empty. The parts the first release
ships are cut before it. Of the order below this much binds: the contract's crates are cut
first, the CLI before the client library, and the vault last. The relay's cut waits on the
first alone.

1. The contract's crates, into this repository, after the first release's changes to the wire
   have landed in `kotton`. Until this cut a request's RFC still lands here first, and its
   types and what answers it are one pull request in `kotton`.
2. The relay, into `kotton-relay`.
3. The CLI, into `kotton-cli`, once the client library's owed list is empty as well as its
   own. Until the library's cut it takes the library from `kotton`, by a pinned commit: the one
   dependency pinned to a repository other than this one, which brings the library and the
   contract and nothing of the system's, and across which no tag is made.
4. The client library, into this repository.
5. The vault, into `kotton-vault`, after the RFC that says what a vault needs to know.

From a cut until this repository's first tag, a dependent pins the commit, as `VERSIONING.md`
says of a commit between tags. The tags are the first release's last step, after every cut
above and after the release's lines have been run from archives built from checkouts: this
repository's tag, then each implementation's, then the distribution's, made after RFC 0003's
test of the release has passed from the release's own archives. A repository the first release
ships nothing from is cut after it.

A native guest is a process the system's sandbox host spawns and confines: it is given its
channel to the daemon at spawn and nothing else, no key, no store, no network. What crosses
that channel, and what the guest may assume, is one contract the protocol defines and holds for
every guest; no guest carries its own cage, and the check that a guest's code reaches nothing
else is the host's, not the guest's.

Repositories are named for the surface or the role, never for the technology that delivers
them. `cli`, `desktop` and `portal` name surfaces a person uses; `gateway`, `relay`, `vault`,
`compositor` and `infer` name roles a process plays; `protocol`, `sdk` and `dist` name what an
implementer, an app developer or an installer takes; `ahead` names the reference tree's role,
the whole in one tree; `kernel` names the role reserved for the last implementation.
The system carries the bare name: it is the node, the root of trust, which every other
repository reaches, implements against, or holds ciphertext for. The rule binds repositories; a
crate inside one may still be named for what it does.

The protocol and the language bindings are Apache-2.0, so that anyone may implement them.
The implementations are AGPL-3.0-or-later, as the trust path is.

## Consequences

- Each repository builds, lints and releases on its own, and a loop can work one without
  touching another. A feature is the next rung of the repository it belongs to.
- The compositor and the desktop are two trust levels, so two repositories: the compositor is
  what makes any desktop safe, and a person installs the one they trust; a desktop is an app,
  and nobody is required to use the reference one. The sandbox host is the system's, so that
  a desktop can be untrusted at all.
- The gateway and the portal are two trust levels, so two repositories: the gateway decides
  what the network may ask; the portal is what a person opens, and it is held to the protocol
  alone, like the desktop and the CLI. The pairing page goes with the portal: it is something
  a person opens, and the door it walks through is the gateway's.
- The protocol is what a message means; the wire format is how it is encoded, versioned under
  the meaning. The protocol carries a version, a changelog naming the RFC for each change, and
  a check that a shipped message's meaning does not change under a version that shipped it.
  That is the price of developing in parallel, and it is what lets anyone write a client, a
  compositor, a desktop, or a vault.
- The vault and the inference runtime are the first tests of the contract: a vault must need
  nothing but the protocol and a disk. What a vault needs to know, and nothing else, is an RFC
  of its own.
- Tooling is not copied repository by repository: the gates every repository runs are one
  small versioned crate and one reusable workflow; the gates that read prose live here, with
  the prose. The integration suite belongs to the distribution and tests archives, since
  "the composed release works" is the distribution's property.
- The word "kernel" is reserved for the kernel. The daemons are the system.

## Reaches

Every repository above. The protocol's own change is this repository's first tag, after the
versioning check exists.
