# Versioning

The protocol is versioned so that every other repository can depend on it by number, and so
that a request's meaning, once shipped, never changes under a version that shipped it.

## The rule

Semantic versioning, below 1.0 for now. Below 1.0, cargo's compatibility boundary is the
minor number, and that is how this repository uses it:

- A **patch** bump adds: a new request, a new field a side may ignore, a new surface. Every
  message that existed before means the same thing and encodes the same way.
- A **minor** bump changes or removes: a message whose meaning or encoding is not what it was.
  A minor bump names, in its changelog entry, the RFC that allowed each change.
- A **major** bump is reserved for 1.0, when the protocol is stable enough that a change to a
  shipped message's meaning is a thing that does not happen.

Every tag is `v<major>.<minor>.<patch>` on `main`, and every change on `main` between tags is
one the next tag will carry.

## Pre-release tags

Two pre-release words exist, and nothing else is used. `vX.Y.Z-dev.N` is a snapshot: a tagged
commit that builds and installs so a pipeline can be proved, promising nothing; any
implementation repository may tag one, and the distribution's lock may name one. `vX.Y.Z-rc.N`
is a candidate for `vX.Y.Z`, and only the distribution tags one, since only the distribution
is what a person installs. Pre-release words compare as text, so `dev` sorts before `rc` and
both before the release. This repository tags no pre-release: a commit between its tags is
pinned by hash, as below.

## Between tags

A dependent that needs a change before it is tagged pins the commit:
`kotton-protocol = { git = "https://github.com/kottonllc/kotton-protocol", rev = "<commit>" }`.
The lockfile records the commit; the pin is moved deliberately and never by `cargo update`.
A commit is never named in a version string: pre-release identifiers compare as text, and a
hash does not order.

## What a binary says

A binary built from a tag reports the tag's version. A binary built from any other commit
reports the next patch as a pre-release with the count and the hash, derived from
`git describe --tags`: `v0.1.0-12-gabc1234` is reported as `0.1.1-dev.12+abc1234`. The part
after `+` is build metadata and never affects ordering. There is no build counter: a release
is built once, from a tag, and its sums name its bytes. If one source is ever built twice
with different bytes, the build's run number goes after the `+` and nowhere else.

## The check

A shipped message's meaning and encoding are held by golden encodings: every request and
reply serialised once per version and committed, so that a change to a shipped message's
bytes fails the build unless the minor bumps. That check lands by its own RFC, and nothing is
tagged `v0.1.0` before it exists.
