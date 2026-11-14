# 0006: One rule for every Ed25519 key and signature

## Context

The protocol's signatures are Ed25519. An identity's log is signed with it, a schema is
published under it (RFC 0004), and a device, a contact, a recovery authority and a release are
each named by such a key. Every one of those rests on two parties agreeing whether a signature
checks.

RFC 8032 does not make them agree. It lets a verifier take the equation with the cofactor or
without it, and it does not say what a verifier does with a key or an `R` of small order, or
with an encoding that is not the canonical one. Libraries have chosen differently, so two
implementations that each follow RFC 8032 can answer differently about the same 64 bytes. Where
they do, a log, a schema or a grant that one holder accepts and another refuses is two records
where the protocol means one.

No RFC says which answer is this protocol's, so an implementation has nothing to agree with but
another implementation's library. RFC 0005 gives the handshake that admits a device and signs
nothing; until it stands at every door, RFC 0002 leaves the QUIC path its mutual TLS, whose
handshake signs under a device's key, and that signature has no rule here either. This page is
the rule.

## Decision

**A key passes, or it is no key.** An Ed25519 public key is 32 bytes that are the canonical
encoding of a point on the curve, and the point is not of small order: multiplied by eight it is
not the identity. An encoding is canonical when its y is less than 2^255 − 19 and it does not
set the sign bit over an x of zero. A key that does not pass is refused where it arrives, in a
message, a log, a store, a list, a certificate or a line a person typed. It is not kept, nothing
is admitted by it, and nothing is checked under it. So one key has one encoding, and a set of
encodings is a set of keys.

**A signature is checked one way.** A signature is 64 bytes, `R` then `S`, under a key `A` over
a message `M`. It checks only if `S`, read least significant byte first, is less than the
group's order; `R` is the canonical encoding of a point not of small order; and
`[S]B = R + [k]A`, where `k` is SHA-512 over `R`, `A` and `M` as the bytes received, reduced by
the group's order. The equation is taken as written, without the cofactor. An implementation may
compute `[S]B − [k]A` and compare its encoding with `R`'s bytes, which is the same check.

**Nothing else is Ed25519 here.** No prehashed form and no context form. Several signatures are
checked one at a time, never as a batch, since a batch cannot show this equation of each.

**A threshold signature is such a signature.** It is checked by this rule and no other, and a
side that assembles one checks it by this rule before it gives it to anyone.

**A transport's handshake is not an exception.** Where a handshake beneath this protocol signs
under a key this protocol names, that signature is checked by this rule. A key this protocol
does not name, a web certificate's or a transport's own, is under that system's rules and admits
nothing here.

**A log names only keys that pass.** An event that sets an identity's key, or carries recovery
authorities, is valid only if every such key it names passes; a holder refuses the event
otherwise. An authority committed and not yet shown is held to this when it is revealed. A
device key an event names that does not pass is not admitted, and the event stands.

**The vectors.** Each is refused, by the clause beside it. An implementation holds every one in
a test.

| What | Bytes, hex | Refused by |
|---|---|---|
| A key: the identity point's encoding | `0100000000000000000000000000000000000000000000000000000000000000` | A key passes: the point is of small order |
| A signature, under any key and over any message: `R` the identity point's encoding, `S` zero | `01000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000` | A signature is checked one way: `R` is of small order |

## Consequences

- Every key made the way RFC 8032 makes one passes, and every signature made the way it makes
  one checks: such a key is a multiple of the base point, which has the group's order, and such
  an `R` is too. The rule refuses nothing an honest side produces.
- A key is checked once, where it arrives, at the cost of three doublings and a comparison. A
  signature costs what the equation costs and no more.
- Because a key has one encoding, a list of admitted keys, an index by key and a `did:key` can
  each be compared as bytes. Two strings never name one key.
- A signature under one key over one message with one `R` has one `S`. The rule does not make a
  signature unique, since a signer may choose another `R`, and nothing here depends on that.
- An implementation whose library checks otherwise checks the rest itself, before or after the
  library's call, or uses another. A library that only checks batches, or only with the
  cofactor, cannot be used for these signatures.
- A side that assembles a threshold signature checks once more than its library asks. For keys
  and commitments of the group's order the two equations agree, so the check refuses nothing
  such a library accepted; it is there so that one rule is the only one.
- A log's event that names a key that does not pass is refused by every holder, so a log never
  resolves to a key under which nothing checks.
- Nothing new crosses. This page adds no message, no field and no value in the clear, and no
  signature is made that was not made before; each observer's row in `design/threat-model.md`
  reads as it did.
- RFC 0005's exchange is untouched: a side still closes on an exchange that yields all zeros,
  and a key in a Montgomery form already admitted is still not admitted.
- Owed to the vectors, each to be generated by the reference and added here by pull request,
  none invented: the other seven encodings of small order; a key whose encoding is not
  canonical; a signature whose `S` is a checking signature's plus the group's order; a signature
  under a key that passes whose `R` is of small order and whose `S` completes the equation; and
  one key, message and signature that pass.
- Left open: whether a later suite replaces Ed25519, which this page neither helps nor hinders.

## Reaches

- `kotton`: the verifying key becomes a type in a crate beneath `kotton-pair` whose construction
  is the key's clause and whose one check is the signature's, and no other crate links the
  signature library (kottonllc/kotton#565); the handshake's signature on the QUIC path, the
  threshold provider's assembled signature and the key log's validation come under it, with the
  vectors above in its tests (kottonllc/kotton#564).
- `kotton-protocol`: this page. No earlier RFC and no page under `design/` is made untrue by it.
  When the type crates arrive here, the key is this type. It ships in the minor version that
  names this RFC.
- `kotton-dist`: nothing.
