# kotton-protocol

The protocol: what crosses between Kotton's processes, and what it means.

**Pre-alpha. Unversioned. No tag yet.** Nothing here is stable, and nothing should be built
against a commit as if it were a release. The first tag follows the first two RFCs.

Kotton exists so that the people who generate data are the people who own it. The system that
makes that true is several processes, each holding exactly the authority it acts on, talking
over channels. This repository is the contract between them: every request and reply, what
crosses with it, what each side must verify, and the surface an app sees. Anyone may
implement it — a compositor, a desktop, a vault, a client — and the reference
implementations exist to prove it, not to be mandatory.

- [`rfcs/`](rfcs/README.md): the decisions that bind more than one repository, and how one lands.
  [RFC 0001](rfcs/0001-the-product-is-repositories-along-process-boundaries-joined-by-the-protocol.md)
  is the shape of the whole.
- [`VERSIONING.md`](VERSIONING.md): how this repository is versioned and depended on.
- [`design/`](design/): the pages that describe the whole as it is — [the principles](design/principles.md),
  [the threat model](design/threat-model.md), [processes and crates](design/processes.md).
- [`papers/`](papers/): what the platform is for ([the ownership thesis](papers/ownership-thesis.md))
  and how it is achieved ([the confinement thesis](papers/confinement-thesis.md)).

The protocol and type crates arrive here from the reference repository after the first
distribution tag, as RFC 0001 says. Until then this repository is prose.

## Licence

The protocol and its bindings are [Apache-2.0](LICENSE), so that anyone may implement them.
The pages and the papers are prose under [CC BY-SA 4.0](papers/LICENSE). Contributions arrive
under the [Developer Certificate of Origin](DCO.md).
