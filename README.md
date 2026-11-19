# kotton-protocol

The protocol: what crosses between Kotton's processes, and what it means.

**Pre-alpha. Unversioned. No tag yet.** Nothing here is stable, and nothing should be built
against a commit as if it were a release. The first tag follows the versioning check.

Kotton exists so that the people who generate data are the people who own it. The system that
makes that true is several processes, each holding exactly the authority it acts on, talking
over channels. This repository is the contract between them: every request and reply, what
crosses with it, what each side must verify, and the surface an app sees. Anyone may
implement it — a compositor, a desktop, a vault, a client — and the reference
implementations exist to prove it, not to be mandatory.

- [`rfcs/`](rfcs/README.md): the decisions that bind more than one repository, and how one lands.
  [RFC 0001](rfcs/0001-the-product-is-repositories-along-process-boundaries-joined-by-the-protocol.md)
  is the shape of the whole.
- [`schemas/`](schemas/README.md): the record schemas of Kotton's core set, one file each.
- [`VERSIONING.md`](VERSIONING.md): how this repository is versioned and depended on.
- [`design/`](design/): the pages that describe the whole as it is — [the principles](design/principles.md),
  [the threat model](design/threat-model.md), [processes and crates](design/processes.md).
- [`papers/`](papers/): what the platform is for ([the ownership thesis](papers/ownership-thesis.md))
  and how it is achieved ([the confinement thesis](papers/confinement-thesis.md)).

The protocol and type crates arrive here from [`kotton`](https://github.com/kottonllc/kotton),
which holds them today, once the first release's changes to the wire have landed there; the
client library follows. RFC 0001 gives the order of the cuts. Until then this repository is
prose and the record schemas, and this is where the code of each repository in RFC 0001's
table is:

| Repository | Where its code is today |
|---|---|
| `kotton-protocol` | the RFCs, the pages and the record schemas here; the crates and the client library in `kotton` |
| `kotton`, `kotton-cli`, `kotton-relay`, `kotton-vault` | in `kotton` |
| `kotton-dist` | in `kotton-dist` |
| `kotton-gateway`, `kotton-portal`, `kotton-compositor`, `kotton-desktop`, `kotton-sdk`, `kotton-infer` | in the reference |
| `kotton-kernel` | reserved, and empty |

## Licence

The protocol and its bindings are [Apache-2.0](LICENSE), so that anyone may implement them.
The pages and the papers are prose under [CC BY-SA 4.0](papers/LICENSE). Contributions arrive
under the [Developer Certificate of Origin](DCO.md).
