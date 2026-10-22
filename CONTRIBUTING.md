# Contributing

This repository is the protocol, and it is public from its first commit. The rules are few,
and they do not bend.

## A change is a pull request

A change to the protocol, or a decision that binds more than one repository, is an RFC under
[`rfcs/`](rfcs/README.md), which says how one lands. A correction to a page, or to the
versioning rule, is a pull request against `main` like any other. The description says what
changed and what checks ran, with their real results. An assessor and a reviewer read it
before it merges.

## Every commit is signed off

Every commit carries a `Signed-off-by: Your Name <email>` line, which `git commit -s` adds.
By it you certify the [Developer Certificate of Origin 1.1](DCO.md): that you have the right
to submit the change under [Apache-2.0](LICENSE) for code and [CC BY-SA 4.0](papers/LICENSE)
for prose, and that the record of it is public and kept.

There is no exemption. A commit an agent made at your direction is authored as you, carries
an `Assisted-by: <tool>` trailer, and carries your own sign-off: an agent cannot certify, and
a commit authored as an agent is refused. A merge commit is a commit: `git merge --signoff`
signs it.

The check that holds this is [`.github/workflows/signoff.yml`](.github/workflows/signoff.yml),
and it runs on every pull request.

## What does not bend

- A merged RFC is never edited; a later one supersedes it.
- A shipped message's meaning never changes under a version that shipped it
  ([`VERSIONING.md`](VERSIONING.md)).
- No repository but this one defines a request.
- A vulnerability is a report to the address in [`SECURITY.md`](SECURITY.md), not an issue and
  not an RFC, until it is fixed and shipped.
