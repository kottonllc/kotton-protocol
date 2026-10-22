# 0001: The product is repositories along process boundaries, joined by the protocol

## Context

One repository held every crate: the daemons that hold keys, the compositor, the sandbox and
the apps it runs, inference, messaging, orgs, the web client, and the tools that build and
check them. The process boundaries were drawn with care, and the crates were grouped by whose
authority they act on; but one workspace is one build, one lint, one release, and a crate one
binary links is a crate every binary pays for. The daemon came to link the sandbox, the
surface, messaging, orgs and inference whether or not a release turned them on; the vault,
which exists to hold ciphertext it cannot read, linked inference; the crate that names what
crosses between processes linked the inference backend. Nothing marked what a tagged release
packed, so no part could be released without the rest, and the work had no order, because
everything advanced at once.

The first cut took the release's tree out of the whole into a repository of its own, proved
to build and test alone. That answered what the first release holds. It left the question of
what everything else is, and what joins the pieces.

## Decision

The product is several repositories, one per process boundary that holds its own authority,
joined by the protocol: this repository, which names every request and reply one process can
make of another, what crosses with it, and what it means; versioned; depended on by number.

| Repository | What it is |
|---|---|
| `kotton-protocol` | the contract: the protocol and type crates; what crosses and any side must verify (the pack envelope, log verification, the admission handshake, the pairing types); the client library; the app surface, defined once, with WIT as its Wasm binding; the handle type that is a file descriptor on today's platforms and a capability later; the RFCs; the pages that describe the whole; the versioning rule |
| `kotton` | the system: the user daemon, the device daemon, the QUIC acceptor, the sync watcher, the device-owner tool, and the sandbox host as a process of its own that holds no keys; ships the system archive |
| `kotton-cli` | the CLI and its REPL mode: unprivileged, built from the protocol and the client library alone and held to that by a check; ships its archive |
| `kotton-relay` | the relay; ships its archive |
| `kotton-vault` | the vault; ships its archive |
| `kotton-compositor` | the reference compositor: the trusted display path, the prompts an app must never draw, input routing; one compositor among any that speak the compositor surface |
| `kotton-desktop` | the reference desktop: the home screen, the dock, the overlays, the first-party apps; an app like any other, one desktop among any |
| `kotton-sdk` | the language bindings of the app surface |
| `kotton-infer` | the inference runtime, the first native guest |
| `kotton-web` | the browser and phone client, the pairing page, the gateway |
| `kotton-dist` | the distribution: which version of each archive goes together, the installers, the launcher, the nine lines, the signed sums over the set |
| `kotton-ahead` | the reference: the whole as it was, read-only, until the last piece is cut from it |
| `kotton-kernel` | reserved: the capability kernel, the last implementation of this protocol; empty until the protocol is stable enough that bare metal is a defined task |

No repository but this one defines a request. A change a dependent needs lands here first, as
a new version; the dependent takes the version. Each repository tags and ships its own
archive; only the distribution says what goes together; a person installs a distribution.

A repository boundary is drawn where there is a versioned contract between the sides. A
crate boundary is drawn where there is an authority boundary. The system's daemons share
crates that are the system's insides, not the contract, and stay in one workspace, each
binary held to the crates it may link.

The protocol and the language bindings are Apache-2.0, so that anyone may implement them.
The implementations are AGPL-3.0-or-later, as the trust path has been.

## Consequences

- Each repository builds, lints and releases on its own, and a loop can work one without
  touching another. A feature is the next rung of the repository it belongs to.
- The compositor and the desktop are two trust levels, so two repositories: the compositor is
  what makes any desktop safe, and a person installs the one they trust; a desktop is an app,
  and nobody is required to use the reference one. The sandbox host is the system's, so that
  a desktop can be untrusted at all.
- "Wire" gives way to "protocol". The protocol is what a message means; the wire format is
  how it is encoded, versioned under the meaning. The pages that say "over the wire" are
  corrected as they move.
- The protocol needs discipline it did not have: a version, a changelog naming the RFC for
  each change, and a check that a shipped message's meaning does not change under a version
  that shipped it. That is the price of developing in parallel, and it is what lets anyone
  write a client, a compositor, a desktop, or a vault.
- The vault and the inference runtime are the first tests of the contract: a vault must need
  nothing but the protocol and a disk, and today it links half the system. What a vault needs
  to know, and nothing else, is the next RFC after the versioning check.
- Tooling is not copied repository by repository: the gates every repository runs are one
  small versioned crate and one reusable workflow; the gates that read prose live here, with
  the prose. The integration suite belongs to the distribution and tests archives, since
  "the composed release works" is the distribution's property.
- The earlier plan, which froze everything the first release did not pack on a branch to be
  revived piece by piece, is replaced by this one. A frozen branch said what was not
  shipping; a repository says what it is.
- The word "kernel" is reserved for the kernel. The daemons are the system.

## Reaches

Every repository above. The protocol's own change is this repository's first tag, after the
versioning check exists.
