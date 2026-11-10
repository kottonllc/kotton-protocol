# 0004: A schema is named by the key that publishes it

## Context

A record is typed by a schema, and a schema is cited by name: in a grant, in the record verbs'
`--schema`, in the record log, and in the table a device's store keeps its rows in. Today that
name is a reverse-domain string, `sh.kotton.secret` or `com.example.notes`, with `sh.kotton.*`
reserved for the schemas Kotton ships. Nothing in Kotton resolves such a name, verifies it or
registers it. It is declared by whoever ships the schema, and anyone may declare any name.

A name grants nothing. A grant is the person's act on an app and a schema, and the records stay
in the person's sealed store, so a stranger who publishes a schema under a bank's name gets no
record of the bank's. What they get is the one place a schema's name is shown to a person: the
grant prompt. "Allow Foo to read your com.bank.statement records" looks authoritative when it is
not, and the person has no way to tell. That is the problem, and it is a problem of what a
prompt may say, not of what a store may hold.

Three answers were weighed. Keep reverse-domain names: they borrow the ownership of a domain
as a convention against collision without checking it, so they favour whoever holds a domain
and let anyone wear a name they do not own. Name a schema by the hash of its definition: a hash
binds the text and nothing else, so every change to a field is a new identity, anyone who
copies the text shares it, and it carries no author, so the prompt still cannot say who shipped
it; it answers collisions, which were never the problem. Name a schema by the key that publishes
it: nobody can publish under a key they do not hold, and a key is the one kind of identity this
design already has, for a device, a contact or a release. The third is this decision.

## Decision

**A schema's identity is `<publisher>/<name>`.** The publisher is the key that publishes the
schema, written as the `did:key` of its public half, the form a device's key and a contact's key
already take. The name is one or more segments of lowercase letters, digits and hyphens,
separated by dots, each beginning with a letter, as a segment of today's names is; an
implementation may bound its length. The publisher signs the schema's definition with that key:
an Ed25519 signature, under the key the `did:key` names, carried beside the definition as
`signature`, base64, never inside the digest or the identity, the form an app manifest's
publisher block already takes. What is signed says what it is: the 16 ASCII bytes
`kotton-schema-v1`, one zero byte, and the definition's 32-byte digest, 49 bytes in that order.
A `did:key` signs more than schemas, and a signature it made over anything else is never a
schema's: an implementation checks the signature over those 49 bytes and nothing else, and
refuses one over a bare digest. The `v1` is the form of the signed message, not a revision of
any schema. A definition whose signature does not check against its publisher is no schema at
all. Nobody can publish under a key they do not hold, so no one can wear another publisher's
name, whatever string they choose for their own.

**A schema's version is the digest of its definition.** The definition is one JSON document:
`publisher`, `name`, `revision`, `fields` and `description`. Its digest is BLAKE3 over the
document serialised by the JSON Canonicalization Scheme (RFC 8785), written as lowercase hex, as
the store digests and writes a chunk. A change to a field is a new revision and a new digest of
the same schema; the publisher states which revisions a reader of a later one may still read,
and nothing infers that from the digests. Where a version must be exact, a record log, a
migration, a grant that pins one, the digest is cited; where a person reads, the revision is
shown.

**A schema is never resolved, verified against a registry, or registered.** It is cited by
identity, declared by whoever ships it, and grants nothing by itself. A grant is the person's
act on an app and a schema. No name is anyone's by right, and no namespace is reserved: the
`sh.kotton.*` strings cease to exist.

**Kotton's own schemas are published under Kotton's release key**, the key that signs a release
(`infra/release/allowed_signers` in the reference). From the first tag on, the files under
`schemas/kotton/` carry that key's `signature`, so an implementation other than the reference
holds Kotton's schemas on the same terms as any publisher's, and inside the reference's release
the release's signature covers them as well. Before the first tag they carry none: nothing
depends on a schema by version yet, the release key never leaves the maintainer's machine, and
the files are what every page here is, a change the maintainer signed off. "Core" means shipped
with the implementation and present on every device, and nothing more: being core gives a
schema no reach another schema lacks and no refusal of its own. Two of Kotton's schemas are
kept from apps, `secret` and `access-log`, and not because they are core: the first release
keeps the keychain and the access log from every app (`schemas/README.md`), and an
implementation holds that refusal for those two identities, by name. A publisher saying the
same of its own schema, in the definition it signs, is a seam this RFC leaves open. A schema
nobody at Kotton has seen works like any other, as RFC 0003 says.

**A prompt shows only names the person, or a source the person chose, gave a key.** It never
names the publisher by a string the publisher chose. The schema's name is shown as its publisher
wrote it, since it says what the records are and claims nothing about who stands behind them;
who stands behind them is said only by the names the person's sources gave the key, or by their
absence. "Allow Foo to read your *statement* records from **My Bank**" appears only if the
person named that key My Bank; otherwise the prompt reads "from a publisher you have not named".
An app is shown the same way, by the key that signed it, and when the app's publisher and the
schema's publisher differ the prompt says so. The first meeting with a publisher's key is where
the person decides what to call it, as they do a contact's; after that, every prompt about that
publisher is quiet.

**A statement about a key is a signed record, from a source the person chose.** Four sources
are named here; the first is built with this decision, and the other three are seams, each one
more source of the same kind of record:

- *A petname*: the person's own name for a key, kept on their devices as a record.
- *An introduction*: a contact's petname for a key, shared by that contact's choice, so a prompt
  can say "Alice and Dad call this publisher My Bank". It is shared as the contact's records are
  shared, by the contact to the people they choose, and never published.
- *A confirmation*: the person's own check of a key against a channel the publisher controls, a
  fingerprint on the publisher's site over TLS, inside their app, printed on a statement,
  recorded with the channel and the date. It trusts that channel once, as a cross-check, never
  as a foundation.
- *A list*: statements about keys published under a key of their own, which a person follows.
  Kotton's release key publishes one, as may anyone.

**A list may inform a grant and never gate it.** A statement a source makes changes what a
prompt shows; it never hides a choice or takes a grant away. A list is fetched whole and read on
the device, so no list learns which publishers a person meets. An implementation ships following
its publisher's list, shown as one list among any and replaceable in one setting; lists that
disagree are shown disagreeing. A prompt names which sources stand behind a name, so what the
person has is what they can see.

## Consequences

- No one can impersonate a publisher: the identity is a key, and the check runs on the person's
  device against a signature, with no party to compromise or compel. Against the ownership bar,
  a schema's identity is shared vocabulary, the same for everyone who uses the schema, so it
  identifies no person and joins nothing; a publisher's key names the publisher, as a release
  key names a releaser, which is the one identity this design intends to be public.
- A stranger with no introduction is shown as exactly that. A store that has checked a
  publisher's papers can say more on a first meeting; this design says less, and says it
  honestly. The gap narrows as the person's contacts and the lists they follow name the
  publishers they meet, and an implementation's own list gives a first user an introduction from
  the start.
- A warning reaches people through the lists they chose, slower than a platform's withdrawal of
  an app from every device at once, by exactly the amount that keeps anyone from silencing a
  publisher alone. There is no kill switch, by design, and the design says so.
- An implementation's default list is real influence over what most people see. That is said
  here, the list is shown as one among any, and the rule that a list informs and never gates is
  what keeps it from becoming the registrar this decision declines to have.
- A confirmation through a publisher's website trusts the web's certificates once, for that
  check. The design does not otherwise trust them, and a confirmation is recorded as what it is.
- The schemas this repository holds move under Kotton's release key: `secret`, `keeps-whole`
  and `access-log`, in `schemas/kotton/`; the rest of the reference's core set follows under the
  reference's issue. A record's URL form becomes `kotton://records/<publisher>/<name>/<id>`.
  Nothing is released, so no install migrates.
- Left open, each a follow-up and not a reason to wait: the record schemas of a petname, an
  introduction, a confirmation and a list entry; the channels a confirmation accepts first; what
  a publisher writes to say which revisions a reader may still read; how a petname and the names
  a person's sources give are shown together when they disagree; and how a list reaches a
  device, which tells the list's publisher no more than fetching a release does and carries no
  identifier of the device or the person.

## Reaches

- `kotton`: `kotton_schemas`'s schema identity type takes the publisher and the name in place of
  a reverse-domain string, and the core set is published under the release key; the record
  store's table name derives from the identity; grants, `schema list|show|register`, the record
  verbs' `--schema`, the record log and the wire carry the identity; the grant prompt is written
  to the rule above; a petname is a record. The reference's issue for this is opened when this
  RFC lands, and the keychain and the access log are built under it.
- `kotton-protocol`: `schemas/` as above; RFC 0003's record and grant wording; the threat
  model's account of what an app can and cannot do with a name.
- `kotton-dist`: nothing until a guide names a schema.
