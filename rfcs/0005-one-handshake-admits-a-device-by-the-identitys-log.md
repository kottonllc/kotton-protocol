# 0005: One handshake admits a device, by the identity's log

## Context

RFC 0002 decided that every connection a device opens to a drive or to another device begins
with a handshake under both keys, defined by the protocol and run the same way over every
transport: an ephemeral exchange keys the channel first, and no durable key crosses outside it.
It left the handshake to be written, and said that until it exists the QUIC path keeps its
mutual TLS. This is that page.

The reference reaches a drive over QUIC with TLS 1.3. Each side presents a certificate whose key
is its own, and the other checks that key against a list. Four things about that are not what
RFC 0002 decided, or not what the principles ask.

- **Identity rides the transport.** A drive reached over a relay's circuit, a link that is not
  QUIC, or a host that ends TLS itself has to carry TLS inside the connection to say who it is.
  And TLS 1.3 shows a server's certificate to whoever connects, so a stranger who finds a
  vault's address learns its durable key.
- **A drive admits a device by a row.** At enrolment a vault keeps one row, for the device that
  enrolled it. Every other device is introduced afterwards by a signed message from a device
  already admitted, sent once to each drive that exists at that moment and ordered by the
  sender's clock. A drive enrolled later is never told of the devices that already exist. A
  drive that was unreachable when a device was withdrawn is never told again, and the withdrawn
  device keeps its door there (the reference's #64). The identity's key log, which the vault
  already holds, could not have told it: the log orders the identity's keys and names no
  device's.
- **The key a device presents is the machine's.** There is one per machine, for every person who
  uses it, and it is shown to every vault. A vault that serves two people who share a machine
  can join them by it.
- **A vault sees where a device is.** The relay's circuit is the last of four ways a device
  tries to reach a drive. The first three connect directly, and a vault sees, and keeps, the
  address a device connects from, at every connection: a record of where the person was.

## Decision

**One handshake, the same over every transport.** It is the Noise Protocol Framework's XK
pattern (revision 34), `Noise_XK_25519_ChaChaPoly_SHA256`, with the ASCII bytes
`kotton-handshake-v1` as its prologue. The side that opens the connection is the initiator. A
side's static key is the Montgomery form of its Ed25519 key, the form a pair handle is already
derived from. The initiator holds the responder's key before it begins, from enrolment for a
drive and from pairing or the identity's log for a device, and the pattern never sends it.

```text
initiator                                   responder
1. -> e, es
2.                                          <- e, ee      payload: the responder's signature
3. -> s, se      payload: the initiator's key, its signature, the identity it acts for,
                 and the latest event of that identity's log it holds
```

Each side signs for itself, as RFC 0002 says. A signature is Ed25519, under the signer's own
key, over the ASCII bytes `kotton-handshake-v1`, one zero byte, one byte that is 1 for the
initiator and 2 for the responder, and the 32-byte handshake hash as it stands when the payload
is written. The initiator's payload carries its Ed25519 public key, and the responder checks
that the key's Montgomery form is the static key the pattern delivered. A signature made here is
bound to this exchange and to the side that made it, and is good for nothing else.

A first message that does not open under the responder's key is not answered, so one who does
not hold a drive's key learns nothing from it, its key included. After the third message the
channel is Noise's transport: frames of at most 65535 bytes, each sealed, read in the order
sent. Nothing is resumed, so every connection meets the admitted set as it stands. A connection
begins with one byte, the handshake's version, which is 1; a responder that does not speak it
answers with the versions it speaks and closes, before any key material crosses.

**What crosses in the clear** is the version byte, each side's ephemeral public value, and
sealed bytes with their tags. No durable key is among them, and no signature over one. The
initiator's key, both signatures, the identity and its log cross only inside the channel, after
the responder has shown it holds the key the initiator expected. A relay, a host that ends TLS,
and a listener on the link see a version, two fresh values and ciphertext.

**A transport carries bytes.** QUIC between a person's own devices, a relay's circuit, a
WebSocket to a vault that cannot accept QUIC, and a link such as Bluetooth each carry the same
frames. On the QUIC path TLS still runs beneath, as QUIC requires, with a certificate made for
the connection and checked by nobody: it carries no identity and proves nothing.

**A device's key is one person's, on one device.** The key a device presents in the handshake is
made for one identity on that device and held as that person's other keys there are held. The
machine's key is not presented to a drive or to another device. Two people who share a machine
share no key a vault sees.

**The identity's log names its devices.** An event of the log that admits a device names the key
that device presents, and an event that withdraws one names the key withdrawn. They are the
events the log already uses to add and remove a device, authorised as the log already requires.
The person's devices, as of any event, are the keys admitted up to it and not withdrawn. The
order of two changes is the log's order, and no clock decides it.

**A drive admits a device by the log, and by nothing else.** In its payload the initiator names
the identity it acts for and the latest event of that identity's log it holds, by sequence
number and digest. Once the channel is up, and before anything else is said, the two sides bring
each other to the later log: the side that is behind is sent the events it lacks and checks them
by the log's own rules, and an event that does not extend the chain it holds is refused. The
responder then admits the initiator if its key is among the identity's devices as of the latest
event the responder holds, and otherwise refuses and closes. A drive keeps no row for a device,
and is never told of one by another.

A responder is checked by what the initiator already holds for it. A drive is the key the device
enrolled it under. Another device is a key among the devices of an identity whose log the
initiator holds, its own person's or a contact's.

**Publication is the connection.** There is no message by which one device introduces another to
a drive, no queue of drives still to be told, and nothing to retry. A device that writes an
event connects to the drives it can reach, and any device's next connection to a drive carries
the log there. A drive enrolled after a device joined learns of that device the first time the
device connects. A withdrawn device is refused by a drive from the first connection any of the
person's devices makes to that drive after the withdrawal.

**A drive on a host is reached through a relay's circuit.** A device does not dial a vault's
address, punch through to it, or ask to be introduced to it directly: the circuit is the path,
not the last resort. So a vault sees the relay's address and never the device's, and the relay,
which sees the device's address, sees a handle and ciphertext and not whose drive it is. Neither
holds both unless one landlord runs both, which the threat model already says. Between devices
the paths stay as they are, and a device that keeps a drive whole is reached as a device.

## Consequences

- The row and the queue are gone, with the three defects that lived in them. A drive enrolled
  late learns each device as it connects. A withdrawal a drive missed reaches it with the next
  connection from any device (the reference's #64, which closes as unnecessary). A device that
  joined can enrol a drive, since a drive no longer asks for the identity's own key (#60). And
  two changes are ordered by the log, not by the clock of whoever sent them (#69).
- A withdrawn device keeps its door at a drive until one of the person's devices next connects
  there, and no longer. This RFC does not shorten that with a clock: a standing that expires has
  to be renewed, and the renewals are a rhythm a host can read and a way for a person who was
  away to be locked out. A drive is as good as the log it holds. Where a withdrawn device could
  still sign for the identity, withdrawing it is not enough, and the person rotates, as RFC
  0003's `device rm` says.
- A landlord learns the keys of the person's devices and the order in which they were admitted
  and withdrawn, as it already does by introduction, and nothing of where they are. Two
  landlords of one person hold the same log and the same device keys, and can tell they serve
  one person. That is true today, since a vault holds the identity's log from enrolment, and
  this RFC does not change it. The seam is a log for each host, derived from the identity's, and
  it is the recovery copy's question as much as the handshake's.
- Every byte to and from a vault crosses a relay. The relay carries more, a relay that is down
  takes the path to every vault with it, and a relay's operator can tell that the far end of
  many busy circuits is a vault, though not whose. Devices keep what they were told to keep, as
  RFC 0002 says. Whether a person may let their devices dial a vault they run themselves is left
  open.
- One who already holds a drive's key can tell that the drive answers at an address. No one else
  can tell that anything there speaks this protocol past the version byte.
- A key signs and takes part in an exchange. The two uses are of one Ed25519 key in its two
  standard forms, the signature is over a message that says what it is and which side made it,
  and nothing either use produces is accepted by the other.
- On the QUIC path two key exchanges run, QUIC's beneath the protocol's, the cost RFC 0002
  accepted. The browser's door keeps its own ceremony, which admits a key a node has not met yet
  by a person's comparison; this handshake is for keys already known.
- Recovery by the phrase is another door, and this RFC leaves it as the threat model describes
  it. One line of that description changes when this is built: a drive refuses a device a
  rotation left out.
- Left open, each a follow-up and not a reason to wait: the bytes of the payloads and of the
  log's device events, which the golden encodings fix; how a drive's own key is replaced;
  rekeying a channel that stays up; padding and timing, which the threat model names as not
  covered; a witness or a standing that expires, should the window above prove too long; and a
  log for each host.

## Reaches

- `kotton-protocol`: the type crates gain the handshake's messages and the log's device events;
  the tenant-device types of #20 are not moved in, since the message they carry ends. The threat
  model's rows for a landlord, a relay's operator and a person holding the phrase change when
  the system does, and the processes page's transports with them. In this pull request the
  landlord's row is corrected for what is already true: it holds the identity's log and the keys
  of the tenant's devices. It ships in the minor version that names this RFC.
- `kotton`: the handshake over QUIC and over a circuit, in place of mutual TLS as the bearer of
  identity; a device key for each person on a device; the log's events naming device keys;
  admission by the log at a vault and at a device, with the row for a tenant's device and the
  message that introduced one removed; and the circuit as the path to a vault. The reference's
  issue is opened when this lands, and #64 closes under it.
- `kotton-relay`: it does what it does now, and carries more.
- `kotton-vault`: it admits by the log, accepts the handshake over whatever it can listen on,
  and is reached through a relay.
- `kotton-gateway`, `kotton-portal`, `kotton-cli`, `kotton-dist`: nothing.
