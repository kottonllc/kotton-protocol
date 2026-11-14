# 0006: One rule for every Ed25519 key and signature

## Context

The protocol's signatures are Ed25519. An identity's log is signed with it, a schema is
published under it (RFC 0004), and a device, a contact, a recovery authority and a release are
each named by such a key. Every one of those rests on two parties agreeing whether a signature
checks.

RFC 8032 does not make them agree. Its decoding (§5.1.3) is exact: 32 bytes whose y is not less
than 2^255 − 19, or that set the sign bit over an x of zero, or that have no x, decode to
nothing. Its verification (§5.1.7) is not: it gives the equation with the cofactor and says the
one without is sufficient, so a verifier may take either, and for some signatures the two
differ. And it says nothing of a key or an `R` that decodes to a point of small order, or to any
point outside the subgroup its keys are made in. Libraries have chosen differently on each, and
not every library decodes as §5.1.3 says. So two implementations can answer differently about
the same 64 bytes, and a log, a schema or a grant that one holder accepts and another refuses is
two records where the protocol means one.

No RFC says which answer is this protocol's, so an implementation has nothing to agree with but
another implementation's library. RFC 0005 gives the handshake that admits a device and signs
nothing; until it stands at every door, RFC 0002 leaves the QUIC path its mutual TLS, whose
handshake signs under a device's key, and that signature has no rule here either. This page is
the rule.

## Decision

Whatever this page does not state is RFC 8032 §5.1: the curve, the base point `B`, the group's
order, the encodings, and how a key and a signature are made.

**Decoding is RFC 8032 §5.1.3's, and nothing laxer.** For a key and for `R` alike, 32 bytes
decode to a point only as §5.1.3 says: bytes whose y is not less than 2^255 − 19, bytes that set
the sign bit over an x of zero, and bytes that have no x are no point, and are refused before
anything else is asked of them. A side whose library reduces such a y, or ignores such a sign
bit, makes this test itself. And where this page says a point is not the identity, the point is
tested, never its bytes against one encoding: the identity is the one point of the subgroup that
a laxer decoder reaches from other bytes, three other encodings, and a side that compared bytes
would take each of them for some other point.

**A key is a point of the prime-order subgroup, canonically encoded, or it is no key.** An
Ed25519 public key is 32 bytes that decode as RFC 8032 §5.1.3 decodes them, to a point that is
not the identity and that the group's order takes to the identity. Bytes that do not decode, a
point of small order, and a point of the subgroup with a point of small order added to it are
each no key. The line is the subgroup, and not only "not of small order", for three reasons:
every key RFC 8032 makes is in it and no other point is one a signer made by the rule; a
library that holds keys to the subgroup and one that does not would otherwise disagree about a
key; and under such a key the rest of the rule is shorter, below. Every encoding §5.1.3 refuses
is of no point, of a point outside the subgroup, or of the identity, so a side that tests the
subgroup and tests the point against the identity refuses them however its library decodes. So
one key has one encoding, and a set of encodings is a set of keys.

**A value that is no key is never used as one.** Wherever 32 bytes arrive that are to be a key,
in a message, a list, a store, a certificate or a line a person typed, and do not pass, nothing
is admitted by them, nothing is indexed by them, and nothing is checked under them. Where they
are all that arrived, what arrived is refused. Where they are part of something that stands
without them, which is a device value in a log's event, below, they stay in it as bytes.

**A signature is checked one way.** A signature is 64 bytes, `R` then `S`, under a key `A` that
passes, over a message `M`. It checks only if all of these hold: `S`, read least significant
byte first, is less than the group's order; `R` decodes as §5.1.3 decodes it, and the point is
not the identity; and `[S]B = R + [k]A`, where `k` is the 64 bytes of SHA-512 over `R`, `A` and
`M` as the bytes received, read least significant byte first and reduced by the group's order.
The equation is taken as written, without the cofactor. Since `A` is of the subgroup, an `R`
that satisfies it is of the subgroup too, which is why no more is asked of `R` than that it is
not the identity. An implementation may compute `[S]B − [k]A` and compare that point's encoding
with `R`'s 32 bytes: the comparison holds the equation and that `R` is canonically encoded, and
`S`'s range and that `R` is not the identity are checked besides.

**Nothing else is Ed25519 here.** No prehashed form and no context form. Several signatures are
checked one at a time, never as a batch, since a batch cannot show this equation of each.

**A threshold signature is such a signature.** It is checked by this rule and no other, and a
side that assembles one checks it by this rule before it gives it to anyone.

**A transport's handshake is not an exception.** Where a handshake beneath this protocol signs
to show a key this protocol names, the signature is Ed25519 and is checked by this rule, and no
other signature scheme shows such a key. A refusal by this rule at a door is the door's one
refusal: it reads as every other refusal there does (RFC 0005). A key this protocol does not
name, a web certificate's or a transport's own, is under that system's rules and admits nothing
here.

**A log's identity and its authorities are keys that pass.** An event that sets an identity's
key, or carries recovery authorities, is valid only if every such key passes; a holder refuses
the event otherwise. Both are read from what the event's digest covers and from the authorities'
keys it carries, so every holder of the event decides the same.

**A device value that does not pass is no device, and the event stands.** RFC 0005 reads an
event's validity from no device key, and this page keeps that. An event that names a device
value that is no key keeps its place. Its copy is whole as RFC 0005 says, read over the values
as bytes: the root is checked over every value the event names, this one among them, and the
value travels with the event. It is never a key. It is dropped before any of RFC 0005's rules of
admission is read, so it is not admitted, withdraws nothing, and is not one of two keys that
share a Montgomery form. Among keys that pass, a Montgomery form is shared by a key and its
negative and by no other value, so that rule of RFC 0005 reads as it did.

**The vectors.** Each is refused. An implementation holds every one in a test.

| What | Bytes, hex | What it shows |
|---|---|---|
| A key: the identity point's encoding | `0100000000000000000000000000000000000000000000000000000000000000` | No key: it decodes to the identity |
| A signature: `R` the identity point's encoding, `S` zero | `01000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000` | Refused because `R` decodes to the identity. Under a key that passes the equation refuses it as well, so it does not alone show that clause held; the owed signature with this `R` does |

**The vectors owed.** kottonllc/kotton#564 owes them: the reference generates each and adds it
to this page by pull request, and none is invented. No tag of this repository is cut, and that
issue does not close, before they are in the page. Each is stated by what makes it tell one
rule from another: but for those marked, it is accepted by the rule with one clause taken out
and by no other such rule. A marked entry is refused by two clauses, both named, and no vector
can part them.

- Keys. The other seven encodings of points of small order: they decode, and only the subgroup
  refuses them. The canonical encoding of a point of the subgroup with a point of small order
  added: it decodes and is not of small order, and only the subgroup refuses it, which is the
  line this page draws. Thirty-two bytes with no x: only the decoding refuses them. An encoding
  §5.1.3 refuses and a laxer decoder takes to a point other than the identity, which is one
  whose y is not less than 2^255 − 19, or the point of order two with the sign bit set: marked,
  since every such point is outside the subgroup, so the decoding and the subgroup both refuse
  it. The three other encodings of the identity, which are y of one with the sign bit set, and y
  of 2^255 − 18 with the sign bit and without it: each marked, refused by the decoding and by
  the test that a key is not the identity.
- Signatures, each under a key that passes. One whose `S` is a checking signature's plus the
  group's order: only `S`'s range refuses it. One whose `R` is the identity point's encoding and
  whose `S` completes the equation: only the clause on `R` refuses it. One whose `R` is a point
  of the subgroup with a point of order eight added, canonically encoded, and whose `S` is made
  as RFC 8032 makes it: the equation with the cofactor accepts it, and only the equation as
  written refuses it.
- An `R` that is not canonically encoded and completes the equation, under a key that passes:
  one of the identity's three other encodings as `R`, with the `S` that completes the equation.
  Marked: the decoding refuses it, and so does the test that `R` is not the identity. There is
  no other such `R`. The point the equation asks for is of the subgroup, and the identity is the
  only point of the subgroup with a second encoding.
- What the reference's test of these vectors shows besides: the rule built over a decoder that
  reduces y and ignores the sign bit over an x of zero, with the identity tested by the bytes of
  its canonical encoding, accepts the three other encodings of the identity as keys and the one
  above as `R`, and the rule refuses all four.
- One key, one message and one signature that pass.
- An event: a recovery rotation, whole, with its salt and its root, that names two keys that
  pass, one value of small order and one canonically encoded value outside the subgroup. The
  copy is whole, the event stands, and the devices it admits are the two keys and nothing else.

## Consequences

- Every key made the way RFC 8032 makes one passes, and every signature made the way it makes
  one checks: such a key is a multiple of the base point other than the identity, and such an
  `R` is too. The rule refuses nothing an honest side produces.
- A key is checked once, where it arrives, at the cost of a decompression and one scalar
  multiplication. A signature costs what the equation costs and no more.
- Because a key has one encoding, a list of admitted keys, an index by key and a `did:key` can
  each be compared as bytes. Two strings never name one key.
- A signature under one key over one message with one `R` has one `S`. The rule does not make a
  signature unique, since a signer may choose another `R`, and nothing here depends on that.
- The reduction of `k` cannot be seen: under a key of the subgroup, `k` and `k` reduced give one
  point. An implementation that does not reduce checks the same.
- An implementation whose library checks otherwise checks the rest itself, before or after the
  library's call, or uses another. A library that only checks batches, or only with the
  cofactor, cannot be used for these signatures.
- A side that assembles a threshold signature checks once more than its library asks. Where the
  key and `R` are both of the subgroup the equation with the cofactor and the one without agree,
  so the check refuses nothing that a library holding both to the subgroup accepted; it is there
  so that one rule is the only one.
- An event that sets an identity's key or carries an authority that does not pass is refused by
  every holder, so a log never resolves to an identity's key, or commits an authority, under
  which nothing checks. An event is not refused for a device value: that value admits no device
  at any holder, and holders shown the same events hold the same chain and admit the same
  devices.
- This page adds no message, no field and no value in the clear, and no signature is made that
  was not made before; it decides which are accepted. What crosses where a transport's handshake
  signs is what RFC 0005's Context says of mutual TLS, and this page does not change it: a
  certificate holding a device's durable key, and a signature under that key over the
  handshake's transcript, go from the listener to whoever connects and from the one who dials to
  the listener it accepted, inside TLS's encryption, so one on the path between sees neither.
  The rows of `design/threat-model.md` this touches are a vault's landlord's, which already
  holds the keys of the devices it was told of and is the side such a signature is shown to, and
  a relay's operator's, which carries such a handshake with its certificates encrypted and
  learns no durable identity from it; both read as they did. That model has no row for one who
  connects to a listener, and kottonllc/kotton-protocol#73 owes it. RFC 0005's handshake, which
  signs nothing and shows no key to one who does not hold the responder's, is what ends this,
  and nothing here delays it.
- RFC 0005's exchange is untouched: a side still closes on an exchange that yields all zeros.
- Left open: whether a later suite replaces Ed25519, which this page neither helps nor hinders.

## Reaches

- `kotton`: the verifying key becomes a type in a crate beneath `kotton-pair` whose construction
  is the key's clause and whose one check is the signature's, and no other crate links the
  signature library (kottonllc/kotton#565); the handshake's signature on the QUIC path, the
  threshold provider's assembled signature and the key log's validation come under it, and the
  vectors owed above are generated there and held in its tests (kottonllc/kotton#564).
- `kotton-protocol`: this page, and RFC 0005 corrected in the same pull request, in four
  sentences: the rule for the keys an inception or a recovery rotation names, and the rule for a
  device event that admits a key, each gain the case of a value that is no key; a whole copy is
  read over the values as bytes; and the Consequences line on what no holder refuses an event
  for names that case too. No page under `design/` is changed. It ships in the minor version
  that names this RFC.
- `kotton-dist`: nothing.
