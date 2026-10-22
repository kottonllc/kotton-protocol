# Principles

## Preamble

Kotton is built to establish a different order of things: one in which **the people who
generate data are the people who own it**, and in which software earns trust through its
construction rather than its declarations. Everything else, the cryptography and the sandbox
and the way the code is laid out, is there to make that sentence true of a real person on a
real machine. The preamble is not a principle and does not by itself govern decisions: it
states the intent the principles serve, and it may be clarified but not narrowed.

## The principles

The rules a change must not break. Each is stated as what must stay true, with where it is
held and its state. "Enforced" means a build check fails when it stops holding. "Labelled"
means it is a stated promise with no check, and says so. "False" means the code does not keep
it today and the gap is named.

A release is blocked while any structural promise is in a third state: true only by
inspection, with nothing checking it.

| № | Principle | What it means | Held by | State |
| --- | --- | --- | --- | --- |
| I | **Ownership is structural** | A cryptographic proof is ownership, not a policy. No Kotton service may read a user's plaintext. | the store's design; the vault's no-plaintext check | design holds; no single gate covers the principle |
| II | **The node is the root of trust** | Keys are generated, data decrypted and grants issued at the user's node. A host provides uptime; the user provides the keys. | a check that no grant envelope is accepted as authorisation on the device daemon | enforced |
| III | **Identity is self-sovereign** | There are no user databases in Kotton, anywhere. | a check that no SQL in the workspace creates a table named for users, accounts, logins or credentials, or a column whose name holds `password`, `passwd`, `passphrase`, `pw_hash`, `pin_hash`, `secret_hash`, `token_hash`, `email` or `phone`, but two columns the check names with their reasons; a check that no type in `core/` or the relay has a field whose name holds one of those, but the fields the check names with their reasons, and that a type named for an account has a `did` field | enforced |
| IV | **Access is explicit and revocable** | No privileged tier for first-party software. Past copies are unaffected by revocation, a known limit. | a check that no admin, root or system capability target exists | enforced |
| V | **Each party owns its copy** | In a conversation, each side owns what it holds; redaction is a marker, not erasure of the other side's copy. | the protocol | labelled |
| VI | **Published is not sent** | A community index aggregates records it was granted and does not own them; anyone granted them can rebuild it. | nothing | labelled; the rebuild has no mechanism yet |
| VII | **Custody is not governance** | An operator may not read, nor accumulate a record of behaviour. A single observation is exhaust; a linked history is property. | design review | labelled; no mechanical check |
| VIII | **One sandbox for everyone** | Every app is a confined component, no app reaches the network, and this applies to us. | a check that the sandbox adds WASI to a linker, an app's or the fetcher's, only in `add_wasi_without_sockets`, which adds exactly the interfaces the check lists, and that nothing in `core/wasm` adds all of WASI at once or imports a WASI path by a glob; a check that the contract's app and occupant worlds import only the interfaces they name today and reach no other through a `use`; and, of what the project's own apps declare, a check that no crate under `guest/apps` declares or resolves a crate on a list of network crates or names `std::net`, `tokio::net` or the WASI socket and HTTP interfaces, and a check that every capability a bundled app's manifest declares links an interface the app world imports, or none, a manifest whose capabilities cannot be read refused | enforced |
| IX | **Data outlives applications** | Your data belongs to the schema, not to the app that wrote it. | the record model | labelled |
| X | **Offline delivery is the node's** | The platform does not buffer messages; a relay that carries is not a buffer; a vault that holds for its tenant is the tenant's. | the relay's zero-lifetime transit | labelled |
| XI | **Software must be auditable** | Everything in the trust path is open source, and may only be strengthened. | the licence check | the licence is checked; the principle is not |
| XII | **No silent changes to the pipeline** | Updates are visible and builds reproducible. | signing on the owner's machine | labelled; reproducibility is not yet checked |
| XIII | **Exit is preserved** | Leaving requires the cooperation of no party other than the user. | the replication model | **false for a hosted node**: a host that runs your daemon can decline to start it, leaving you keys to ciphertext you cannot reach until you move it |

The preamble and these thirteen are the project's constitution, and this page is its text. A
build check (`cargo xtask constitution-check`) reads this table and holds the principles whose
State is `enforced`; it refuses when the table marks one it does not inspect, or it inspects
one the table does not mark. Code that cites a principle cites it by its number here. The
constitution is a draft: it has not been ratified by anyone but its author, and how it would be
amended has not been designed.
