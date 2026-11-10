# 0005: One handshake admits a device, by the identity's log

## Context

RFC 0002 decided that every connection a device opens to a drive or to another device begins
with a handshake under both keys, defined by the protocol and run the same way over every
transport: an ephemeral exchange keys the channel first, and no durable key crosses outside it.
It left the handshake to be written, and said that until it exists the QUIC path keeps its
mutual TLS. This is that page.

The reference reaches a drive over QUIC with TLS 1.3. Each side presents a certificate whose key
is its own, and the other checks that key against a list. Three things about that are not what
RFC 0002 decided, or not what the principles ask.

- **Identity rides the transport.** A drive reached over a relay's circuit, a link that is not
  QUIC, or a host that ends TLS itself has to carry TLS inside the connection to say who it is.
  And TLS 1.3 shows a server's certificate to whoever connects, so a stranger who finds a
  vault's address learns its durable key.
- **A drive admits a device by a row.** At enrolment a vault keeps one row, for the device that
  enrolled it. Every other device is introduced afterwards by a signed message from the device
  that holds the identity's key, sent once to each drive that exists at that moment and ordered
  by the sender's clock. A drive enrolled later is never told of the devices that already exist.
  A drive that was unreachable when a device was withdrawn is never told again, and the
  withdrawn device keeps its door there (the reference's #64). The identity's key log, which the
  vault already holds, could not have told it: its events carry the identity's keys and its
  recovery, and name no device.
- **The key a device presents is the machine's.** There is one per machine, for every person who
  uses it, and it is shown to every vault. A vault that serves two people who share a machine
  can join them by it.

The path a connection takes is a fourth matter and is not decided here. A device dials a vault's
address before it tries a relay's circuit, so a vault sees the address a device connects from.
Whether the circuit should be the path to a vault is the next RFC's (#48).

## Decision

**One handshake, the same over every transport.** It is the Noise Protocol Framework's XK
pattern (revision 34), `Noise_XK_25519_ChaChaPoly_SHA256`. The side that opens the connection is
the initiator. A side's static key is the Montgomery form of its Ed25519 key, the form a pair
handle is already derived from. The initiator holds the responder's key before it begins, from
enrolment for a drive and from pairing or the identity's log for a device, and the pattern never
sends it.

```text
initiator                                   responder
0. -> the version, one byte
1. -> e, es
2.                                          <- e, ee
3. -> s, se      payload: the identity the initiator acts for, the latest event of
                 that identity's log it holds, and the versions it speaks
```

A connection begins with one byte, the handshake's version, which is 1, and the prologue is the
ASCII bytes `kotton-handshake-` followed by that byte, so the version is bound into every key.
An initiator opens with the latest version it speaks, and with an earlier one only after that
went unanswered. Each handshake message is one frame, its length first as two bytes, most
significant first. A responder that does not speak the version, or holds no key the first
message opens under, reads that one frame and closes, and says nothing either way; one that
holds several keys tries each. Either side closes if an exchange yields all zeros, which is what
refuses a key of low order. After the third message the channel is Noise's transport: frames of
at most 65535 bytes, each sealed, read in the order sent. The responder's first frame, to any
initiator that completed the handshake, states the versions it speaks, and a side that finds
both speak a later version than the one in use closes. Nothing is resumed, so every connection
meets the admitted set as it stands.

Neither side signs anything. The pattern itself shows each side that the other holds the static
key it claims: the responder's by the first exchange, the initiator's by the last. The responder
takes the initiator's Ed25519 key to be the one, among the devices the log admits, whose
Montgomery form is the static key the pattern delivered. A signature over the exchange would add
only a proof a third party could check that the two had met, which is a record this design does
not want made.

**What the handshake gives, and to whom.** One who connects without the responder's key, and has
not recorded a first message sent to it, is told nothing: not its key, and not that anything at
the address speaks this protocol. A listener on the path sees the version byte and the lengths
of three frames, which is enough to guess the protocol and says nothing of who. One who holds
the responder's key, or has such a recording, can tell that the responder answers there, and no
more. The initiator's key is shown only to the responder it named, never to one standing
between, active or passive. A key stolen later opens no recorded connection. And one who steals
a side's key can answer as that side, and not as the other to it.

**What crosses in the clear** is the version byte, each side's ephemeral public value, each
frame's length, and sealed bytes with their tags. No durable key is among them, and no
signature. The initiator's key, the identity and its log cross only inside the channel, after
the responder has shown it holds the key the initiator expected. A relay's operator, a host that
ends TLS, and a listener on the link see a version, two fresh values, lengths and ciphertext.

**A transport carries bytes.** QUIC, a relay's circuit, a WebSocket to a vault that cannot
accept QUIC, and a link such as Bluetooth each carry the same frames. On the QUIC path the
frames are one stream, and TLS still runs beneath, as QUIC requires, with a certificate made for
the connection and checked by nobody, no protocol name and no ticket: it carries no identity and
proves nothing.

**A device's key is one person's, on one device.** The key a device presents in the handshake is
made for one identity on that device and held as that person's other keys there are held. The
machine's key is not presented to a drive or to another device. Two people who share a machine
share no key a vault sees.

**The identity's log names its devices.** The log gains one kind of event, a device event. It
acts under the identity's current keys and does not change them, and it names one change: a key
admitted, or a key withdrawn. The inception names the key of the device it is made on. A
recovery rotation names every device that stands after it, and a key it does not name is
withdrawn by it. The digest of each of these covers the keys it names, so the identity's
identifier, which is its inception's digest, covers its first device's key. No other event
changes the devices, and an event that would leave two admitted keys with one Montgomery form is
not valid. So the person's devices, as of any event, are the keys named by the last recovery
rotation up to it, or by the inception if there is none, with each device event since applied in
the log's order. No clock orders two changes. Who may sign for the identity is the log's rule
and is not changed here; in the reference today that is the device that holds the identity's
key.

**One chain, and how two are settled.** Two chains part at the first event they do not share. A
chain that holds a recovery rotation at or after that event stands over one that holds none.
Where neither holds one, the chain whose first unshared event has the lower digest, as bytes,
stands. Where both hold one, a side keeps the chain it holds. A side that holds the chain that
does not stand replaces it, from the parting on, when it is shown the one that does. So, except
between two recovery rotations, every drive and device that is shown both chains settles on one,
in whatever order they arrived. A device whose event was replaced writes its change again and
tells its person, and a responder that replaces part of its chain tells an initiator it admits.

**A drive admits a device by the log, and by nothing else.** A drive holds an identity's log
from enrolment, when the enrolling device gives it, and keeps no event for an identity it holds
no log for. In its payload the initiator names the identity it acts for and the latest event of
that identity's log it holds, by sequence number and digest. The responder checks the
initiator's key against the log it holds for that identity. If the log admits the key, the
responder's first frame says so and names the latest event it holds, and each side then sends
the other the events of its chain the other lacks, from where the two part; a responder whose
log then withdraws the initiator closes. If the log does not admit the key, or the responder
holds no such log, its first frame asks, in one form whatever the reason, for the events the
initiator holds, and the initiator sends the latest of them in one frame, as many as the frame
holds and at most 64, or an empty frame if it holds none. The responder keeps those that extend
or stand over what it holds and checks again. If the key is now admitted it says so, as above;
if not, it closes. It closes in one way, and to an initiator it has not admitted it sends no
event and does not say whether it holds the identity or where its log stands, except to a
recovery claim, below. A device whose admission lies further past a drive's log than 64 events
is admitted there once a device the drive admits has connected. A drive keeps no row for a
device, and is never told of one by another.

A responder is checked by what the initiator already holds for it. A drive is the key the device
enrolled it under. Another device is a key among the devices of an identity whose log the
initiator holds, its own person's or a contact's.

**Recovery keeps the door it has.** A device that holds only a recovery authority, the phrase or
a guardian's, is not among the identity's devices, and cannot write the rotation that makes it
one without the log's latest event and the sealed copy a drive keeps. A drive answers its claim
inside the channel. The reference answers it today with the whole log and the sealed copy; what
the claim proves, and how little it can open before a rotation is written, is not decided here
(#49). Until it is, whoever can make the claim learns that the drive keeps the identity.

**Publication is the connection.** There is no message by which one device introduces another to
a drive, no queue of drives still to be told, and nothing to retry. A device that writes an
event connects to the drives it can reach, and any device's next connection to a drive carries
the log there. A drive enrolled after a device joined holds that device from enrolment, since
the log it is given names it. A withdrawn device is refused by a drive from the first connection
made there by a device that holds the withdrawal.

## Consequences

- The row and the queue are gone, with the three defects that lived in them. A drive enrolled
  late holds every device from enrolment. A withdrawal a drive missed reaches it with the next
  connection from a device that holds it (the reference's #64, which closes as unnecessary). A
  device that joined is admitted by a drive, which no longer asks for the identity's own key
  (#60). And two changes are ordered by the log, not by the clock of whoever sent them (#69).
- A withdrawn device keeps its door at a drive until a device that holds the withdrawal connects
  there, and no longer. RFC 0002 and RFC 0003 said "at its next connection"; they are corrected
  to say so. This RFC does not shorten the wait with a clock: a standing that expires has to be
  renewed, and the renewals are a rhythm a host can read and a way for a person who was away to
  be locked out. A drive is as good as the log it holds.
- A withdrawn device that can still sign for the identity can write an event of its own. Where
  its chain parts from the person's the lower digest decides, so it may stand, until the person
  writes a recovery rotation: a chain that holds one stands over a chain that holds none. A
  rotation that is not a recovery rotation retires no key against its holder, who can part the
  chain beneath it. Nor does a recovery rotation end it where that holder moved first: one who
  can sign for the identity can add a recovery authority of its own and rotate under it, and
  then both chains hold one and each holder keeps the chain it was shown first. RFC 0003's
  `identity rotate` is what a person has against a device in someone else's hands, and it is a
  race, which this page does not win for them (#50).
- Two holders of a recovery authority who each write a rotation are not settled by the rule:
  each drive and device keeps the chain it was shown first, and they may differ. A phrase two
  people hold has two owners, as the threat model says of it.
- Whoever holds a recovery authority can shut every device out of every drive at once, by a
  recovery rotation that names only their own. The reference acts on a recovery rotation as soon
  as it sees one, and the contest window an event carries is still not enforced. This is a power
  that holder did not have at a drive before: the threat model says a recovery unpairs no device
  at a node, and that paragraph changes when this is built. It is also what taking an identity
  back from a thief requires, where the thief has not moved first.
- A landlord learns the whole history of the person's devices, each key and the order in which
  it was admitted and withdrawn, from the first. Before, it learned the devices introduced to
  it. Two landlords of one person hold the same log and the same device keys, and can tell they
  serve one person. That is true today, since a vault holds the identity's log from enrolment. A
  contact who holds the person's log, to know their devices by it, learns the same history. The
  seam is a log for each holder, derived from the identity's, and it is the recovery copy's
  question as much as the handshake's.
- A device that has not been shown a withdrawal still answers the withdrawn device, and still
  dials it. The wait above is every holder's, not only a drive's.
- A key takes part in an exchange in its Montgomery form and signs in its Ed25519 form
  elsewhere, and a pair handle already uses the first form between two static keys. The known
  analyses of one key in both uses hold while an exchange's output goes only into the key
  derivation and is never released, which is true of every use here.
- RFC 0002 called the handshake the signed form of the ceremony the browser's door runs. It is
  not that. The ceremony shows the initiator's key to whoever completes the ephemeral exchange,
  since neither side knows the other yet; here the initiator already holds the responder's key,
  so the first message is bound to it and neither key is shown to anyone else. RFC 0002's
  sentences are corrected. The browser's door keeps its ceremony, which admits a key a node has
  not met by a person's comparison, and pairing keeps its own version in its own first frame,
  which this RFC does not change.
- On the QUIC path two key exchanges run, QUIC's beneath the protocol's, the cost RFC 0002
  accepted.
- Left open, each a follow-up and not a reason to wait: the path to a vault, which is #48;
  recovery's claim, which is #49; whether a recovery rotation stands when its authority was
  added after the parting, which is #50; the bytes of the payload, of the frames that follow it
  and of the log's events, which the golden encodings fix; what a drive keeps of what a device
  wrote while an event that admitted it stood and was then replaced; how a contest window bears
  on a recovery rotation's reset of the devices; a device whose admission lies more than 64
  events past a drive's log and has no other device to go first; how a drive's own key is
  replaced; rekeying a channel that stays up; padding and timing, which the threat model names
  as not covered; whether a responder should remember first messages so a recording tells
  nothing; a witness or a standing that expires, should the wait above prove too long; who
  besides the first device may sign for the identity; a version for the pairing payload, which
  #12 asked of this page and which is pairing's to state; and a log for each holder.

## Reaches

- `kotton-protocol`: the type crates gain the handshake's messages and the log's device event;
  the tenant-device types of #20 are not moved in, since the message they carry ends. In this
  pull request: RFC 0002's paragraph on the handshake and RFC 0003's line for `device rm` are
  corrected as above; the threat model's row for a vault's landlord is corrected for what is
  already true, that it holds the identity's log, the keys of the devices it was told of, and
  the address a device dials it from; and `rfcs/README.md` asks of an RFC what #12 asked of this
  one. The threat model's rows for a landlord, for a person holding the phrase and for someone
  with your device, and the processes page's transports, change when the system does. It ships
  in the minor version that names this RFC.
- `kotton`: the handshake over QUIC and over a circuit, in place of mutual TLS as the bearer of
  identity; a device key for each person on a device; the log's device event, the inception and
  the recovery rotations naming keys, and the rule for two chains; admission by the log at a
  vault and at a device, with the row for a tenant's device and the message that introduced one
  removed. Nothing is released, so no identity or log is carried over: the inception's digest
  changes with what it names. The reference's issue is opened when this lands, and #64 closes
  under it.
- `kotton-vault`: it admits by the log and accepts the handshake over whatever it can listen on.
- `kotton-cli`: `device rm` says what the wait is and when to rotate, and `status` shows a drive
  whose log is behind.
- `kotton-relay`, `kotton-gateway`, `kotton-portal`, `kotton-dist`: nothing.
