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
handle is already derived from, or at a contact's door the key the pair derives for the hour.
The initiator holds the responder's key before it begins, from enrolment for a drive, or from
the line printed beside a recovery phrase where it holds no more than the phrase, from pairing
or the identity's log for a device, and from the pair's own derivation for a contact's door, and
the pattern never sends it.

```text
initiator                                   responder
0. -> the version, one byte
1. -> e, es
2.                                          <- e, ee
3. -> s, se      payload: the identity the initiator acts for, or none, the latest
                 event of that identity's log it holds, or none, and the versions
                 it speaks, padded to one length
```

A connection begins with one byte, the handshake's version, which is 1, and the prologue is the
ASCII bytes `kotton-handshake-` followed by that byte, so the version is bound into the
handshake's hash, under which every message is sealed. An initiator opens with the latest
version it speaks, and with an earlier one only after that went unanswered. Each handshake
message is one frame, its length first as two bytes, most significant first. A responder that
does not speak the version, or holds no key the first message opens under, reads that one frame
and closes, and says nothing either way; one that holds several keys tries each. A responder
waits 10 seconds, counted from when it accepts the connection, for the version byte and the
whole of the first frame, and closes on a connection that has not sent them by then, having
written nothing. The wait is one value at every responder, and it is written beside the golden
encodings. Either side closes if an exchange yields all zeros, which is what refuses a key of
low order. After the third message the channel is Noise's transport: frames of at most 65535
bytes, each sealed, read in the order sent. The responder's first frame, to any initiator that
completed the handshake, states the versions it speaks, and a side that finds both speak a later
version than the one in use closes. Nothing is resumed, so every connection meets the admitted
set as it stands.

Neither side signs anything. The pattern itself shows each side that the other holds the static
key it claims: the responder's by the first exchange, the initiator's by the last. The responder
takes the initiator's Ed25519 key to be the one, among the keys it admits there, whose
Montgomery form is the static key the pattern delivered. A signature over the exchange would add
only a proof a third party could check that the two had met, which is a record this design does
not want made.

**What the handshake gives, and to whom.** To one who connects without the responder's key, and
has not recorded a first message sent to it, a responder sends no byte of this protocol: not its
key, and nothing that says the protocol is spoken there. Every such refusal is the same, one
frame read, or the wait run out, and the connection closed. What the transport beneath has shown
by then is said with what crosses in the clear. A listener on the path sees the version byte and
the lengths of three frames, which is enough to guess the protocol and says nothing of who. One
who holds the responder's key, or has such a recording, can tell that the responder answers
there, and no more. The initiator's key is shown only to the responder it named, never to one
standing between, active or passive. A key stolen later opens no recorded connection. And one
who steals a side's key can answer as that side, and not as the other to it.

**What crosses in the clear** is the version byte, each side's ephemeral public value, each
frame's length, and sealed bytes with their tags. No durable key is among them, and no
signature. The initiator's key, the identity and its log cross only inside the channel, after
the responder has shown it holds the key the initiator expected. So does a recovery claim: the
authority's public key and a tag go to the drive whose key the device began with and to no one
between, nothing in it is signed, and the path sees a sealed frame where an initiator's events
would be and one more for each log that answers. A relay's operator, a host that ends TLS, and a
listener on the link see a version, two fresh values, lengths and ciphertext. The transport
beneath shows what it shows to whoever reaches it, before any frame: on the QUIC path the
listener completes TLS with anyone, so one who connects learns that something there accepts
QUIC, and on a relay's circuit an open tells the one who opens it that a reservation stands.

**A transport carries bytes.** QUIC, a relay's circuit, a WebSocket to a vault that cannot
accept QUIC, and a link such as Bluetooth each carry the same frames. On the QUIC path the
frames are one stream, and TLS still runs beneath, as QUIC requires, with a certificate made for
the connection and checked by nobody, no protocol name and no ticket: it carries no identity and
proves nothing. Pairing's socket presents such a certificate too, and the key each side of a
ceremony learns of the other crosses inside the ceremony's sealed channel, by the frames the
ceremony already runs where there is no certificate; its frames and its version are not changed.
A process that accepts a transport's connections and holds no key carries the frames and answers
nothing: the handshake is answered by the process that holds the key.

**A device's key is one person's, on one device.** The key a device presents in the handshake to
a drive or to another of its person's devices is made for one identity on that device and held
as that person's other keys there are held. At a contact's door it presents the pair's key for
the hour and no key of its own. The machine's key is not presented to a drive or to another
device. Two people who share a machine share no key a vault sees.

**The identity's log names its devices.** The log gains one kind of event, a device event. It
acts under the identity's current keys and does not change them, and it names one change: a key
admitted, or a key withdrawn. The inception names the key of the device it is made on. A
recovery rotation names every device that stands after it, and a key it does not name is
withdrawn by it. No other event changes the devices. So the person's devices, as of any event,
are the keys named by the last recovery rotation up to it, or by the inception if there is none,
with each device event since applied in the log's order. An event does not admit a key whose
Montgomery form is that of a key already admitted, or of another key the same event names, and
an event that admits a key already admitted, or withdraws one that is not, changes nothing; in
each case the event keeps its place in the log. No clock orders two changes. Who may sign for
the identity is the log's rule and is not changed here; in the reference today that is the
device that holds the identity's key.

**An event's digest covers the keys it names through a root.** An event that names device keys,
which is an inception, a recovery rotation or a device event, carries a device root of 32 bytes
in what its digest covers. The keys it names, the change where it is a device event, and a salt
are carried beside the root, and the digest does not cover them. So the identity's identifier,
which is its inception's digest, covers its first device's key, and an event has one digest
whether or not a copy of it carries the keys.

The root is made with SHA-256. The salt is 32 random bytes, made for the one event by the device
that writes it. Each key the event names gives a leaf. The key's blind is HMAC-SHA256, keyed
with the salt, over the ASCII bytes `kotton-device-blind`, one byte for the change, 1 for a key
admitted and 2 for a key withdrawn, and the 32 bytes of the key. Its leaf is the SHA-256 of the
ASCII bytes `kotton-device-leaf`, the same byte, the key and the blind. The key an inception
names, and each key a recovery rotation names, is a key admitted. A device event names one key,
and its root is that key's leaf. Where an event names more, the leaves, no two of them alike,
are put in ascending order as bytes and joined in pairs, the first with the second and the third
with the fourth, a last one that has no pair carried up as it is. A pair gives the SHA-256 of
the ASCII bytes `kotton-device-node` and its two values, the lower first. The values that result
are joined the same way until one is left, and that one is the root.

A copy of such an event is whole when it carries the salt and what the event names. A drive, and
a device of the person, keep such an event only as a whole copy that names at least one key and
none twice, and whose salt and keys make the root, and they read the devices from it. A
contact's copy carries the root alone, with no salt and no key. It holds the same events under
the same digests and the same signatures, checks each as a drive does but for the keys against
the root, and settles two chains by the same rule, below. Whether an event is valid, and which
chain stands, are read from what its digest covers, from its signature and from the authorities'
keys it carries, and from no device key and no salt. So a holder of the roots alone and a holder
of whole copies, shown the same events, hold the same chain; an event whose keys have not
reached a drive is to that drive an event it has not been shown. Without the salt no key can be
tried against a root, and the salt goes only where the keys go. A key can be shown under a root
by its blind and the values beside its path, which recognise no other key; no page here shows
one.

**One chain, and how two are settled.** A chain that extends another replaces it. Two chains of
which neither extends the other part at the first event they do not share. On each, one recovery
rotation is weighed: the first the chain holds at or after that event. It counts only if the
last event the two chains share already committed the authority that signed it. An authority
added after the parting counts for nothing against a rotation that counts, and no later rotation
is weighed in its place. A chain that holds a rotation that counts stands over one that does
not. Between two that do not, a chain that holds a recovery rotation at or after the parting
stands over one that holds none, and where neither holds one the chain whose first unshared
event has the lower digest, as bytes, stands. Where both hold a rotation that counts, or both
hold only one that does not, a side keeps the chain it holds. A side that holds the chain that
does not stand replaces it, from the parting on, when it is shown the one that does.

What an event committed is read from its commitment, which its digest covers: it committed an
authority if a path from that authority's key reaches the root. An event that sets a commitment
carries the keys under it and is not valid unless they make that root, and a recovery rotation's
digest covers the authority that signed it. So no two holders of an event differ on what it
committed or on who signed it. Only a recovery rotation takes an authority out of the log's
commitment, so a rotation that counts at one parting counts at every later one it still follows,
and standing over never runs in a circle, among any number of chains. So a side that has been
shown each of them, in whatever order, holds one that none of the others stands over, and where
the rule leaves no two of them to the side that holds them, that is the same chain at every
drive and device. A device whose event was replaced writes its change again and tells its
person, and a responder that replaces part of its chain tells an initiator it admits.

**A drive admits a device by the log, and by nothing else.** A drive holds an identity's log
from enrolment, when the enrolling device gives it, and keeps no event for an identity it holds
no log for. In its payload a device names the identity it acts for and the latest event of that
identity's log it holds, by sequence number and digest. The responder checks the initiator's key
against the log it holds for that identity. If the log admits the key, the responder's first
frame says so and names the latest event it holds, and each side then sends the other the events
of its chain the other lacks, from where the two part; a responder whose log then withdraws the
initiator closes. If the log does not admit the key, or the responder holds no such log, or the
payload names no identity and the key is not one the responder admits without a log, below, its
first frame asks, in one form whatever the reason, for the events the initiator holds, and the
initiator answers in one frame that begins with one byte, its kind. Kind 1 is its events: after
the byte, the latest of them, as many as the frame holds and at most 64, or none where it holds
none. Kind 2 is a recovery claim, below. A responder closes on a frame that has no kind byte, or
any other. The responder keeps those events that extend or stand over what it holds and checks
again. If the key is now admitted it says so, as above; if not, it closes. It closes in one way,
and to an initiator it has not admitted it sends no event and does not say whether it holds the
identity or where its log stands, except to a recovery claim, below. A device whose admission
lies further past a drive's log than 64 events is admitted there once a device the drive admits
has connected. A drive keeps no row for a device, and is never told of one by another.

A responder is checked by what the initiator already holds for it. A drive is the key the device
enrolled it under, or the key on the line printed beside a recovery phrase. Another of the
person's devices is a key among the devices their log names. A contact's door is the key the
pair derives for the hour, below.

**Who stands at a door and is in no log.** A vault's operator acts for no identity. Its payload
names none, in the place the identity would be, and the vault admits it by its own set of
operators' keys and never by a log. A device that holds only a recovery authority names none
either, since it cannot name the identity it acts for. A vault tells the two apart by the key
and by nothing in the payload: an initiator that names none and whose key is not an operator's
is asked for its events as any initiator a log does not admit is asked, which is where a claim
is made, and is sent nothing an operator is sent. The payload is padded to one length whoever
sends it, a device, an operator or a device with a claim to make, so the handshake does not tell
the path one from another. A gateway and a co-signer are paired by a person and hold none of
that person's data keys. The log names only devices that hold them, so a gateway and a co-signer
are admitted by the pairing of the person who paired them, at that person's devices and nowhere
else, by the key and with no identity in their payload, and reach no drive. A contact's door
runs this handshake under the keys the pair derives for the hour, the initiator's and the
responder's both, with no identity in its payload. A responder knows the door by the key the
first message opened under, and admits there the key the pair derives for the initiator and no
other. Neither end shows the other a device's key, and a vault that keeps the door for its
person answers there under the hour's key that person's device gave it, as a device would.

**Recovery keeps its door, and the door opens less.** A device that holds only a recovery
authority, the phrase or a guardian's, is not among the identity's devices, and cannot write the
rotation that makes it one without the event its rotation is to extend and the sealed copy a
drive keeps. It holds the drive's key, from the line printed beside the phrase, and it does not
hold the identity, which no phrase derives. So its payload names no identity and no event, the
drive's first frame asks for its events as it asks any initiator a log does not admit, and the
device answers with a frame of kind 2, which carries a claim in place of events: the public key
of the authority it holds, and a tag. The tag is the 32 bytes HKDF-SHA256 yields from an
exchange between the Montgomery form of that key and the drive's ephemeral value of this
handshake, with the handshake's hash, the 32 bytes Noise calls `h` once the third message is
read, as its salt and the ASCII bytes `kotton-recovery-claim` followed by the 32 bytes of that
public key as its info. Nothing is signed, and the drive could have made the tag itself, so it
holds nothing it can show anyone. The hash covers both ephemeral values, the drive's key and the
device's, so a claim answers one connection and is of no use on another, from another device or
to another drive.

The log already lets a drive check it. An event's recovery commitment is a root over the public
keys of the identity's authorities, a key a phrase derives or a key a set of guardians share,
and the event that sets a commitment carries those keys. The drive makes the exchange from its
side, closes as on any refusal if it yields all zeros, and checks the tag. Then it looks in each
log it holds for the event to answer from: the last event that commits the key, by a path to the
commitment itself and not by the list carried beside it, and after which the log holds no
recovery rotation, or holds a first one whose authority that event did not commit. That is the
last event over which a rotation under the key extends the log the drive holds or stands over
it. An event that commits the key and is followed by a first rotation under an authority it also
commits is passed over, since a rotation written there would be left to the side that holds the
log. A claim that passes both is answered with one frame for each such log: the identity, that
event by sequence number and digest, the path that shows the key under that event's commitment,
and the sealed copy. Then the drive closes. A claim that fails either is closed on as any
initiator the drive does not admit is closed on, having been sent what each of them is sent and
no more; and since the tag is checked first, one who does not hold the key it claims learns
nothing from how long the drive took. With the answer the device writes its rotation, which
names its own key, and connects again as a device that holds an event the drive lacks; where
more than one log answered, its own is the one whose sealed copy its phrase opens. Once the
drive holds the rotation the log admits the device, the drive sends it the rest of the log, and
the device checks from the inception what it could not check before.

A guardian's authority is one key under the commitment, so its claim is the same tag, and the
exchange behind it is made by as many guardians as that key takes, while the connection is open;
how they make it together is left to the page that builds guardians. A drive counts no tries and
keeps no record of who made them: a claim is an exchange under a key, a phrase is longer than
can be guessed, and whoever holds the log can test a phrase against it without asking a drive,
so a refusal confirms nothing. A wrong claim costs its maker a connection. A phrase derives its
keys in order and the log commits one of them at a time, so a device that does not know which
makes a claim for each it would try, a connection each, and the one that passes tells it which.

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
  its chain parts from the person's the lower digest decides, so it may stand; and it can add a
  recovery authority of its own and rotate under it, which stands over a chain that holds no
  recovery rotation and shuts the person's devices out, until the person writes a recovery
  rotation. A rotation that is not a recovery rotation retires no key against its holder, who
  can part the chain beneath it. Where the person's rotation is under an authority the log
  committed before that holder added one, and is written over an event from before that
  addition, it counts and the holder's does not, beside it or beneath it, whoever moved first.
  Written over an event that already commits the holder's authority it shares that event with
  them, both count, and each side keeps the chain it held. RFC 0003's `identity rotate` is what
  a person has against a device in someone else's hands, and it is not a race where their phrase
  was committed before that holder's authority and their rotation is written beneath it. Where
  that holder has added an authority and not rotated under it, the event a drive names to a
  device that holds only a phrase already commits that authority, and whether it should name the
  one before the addition is left open (#55).
- What the rule costs is the identity whose recovery authorities were all added by the key that
  was taken: none lies beneath where that key's holder can part the chain, and the rule does not
  help it. The holder parts the chain before the first was added, no rotation counts on either
  side of that parting, and each side keeps the chain it was shown first. Where the holder's own
  authority lies beneath the person's in a history both share, because it was added first or
  because its event had the lower digest and the person's devices wrote theirs again above it,
  the rule is against the person: the holder's rotation counts and theirs does not, at every
  drive, whoever was shown first. What is left to them is to part the chain beneath that
  addition with the identity's key, where first shown is first kept, and a person who holds only
  a phrase cannot do that. In the reference today an identity is made with no authority and
  gains its phrase by an event its own key signs, so this is every identity whose first device
  is taken. An authority the inception commits lies beneath every parting.
- The rule ranks authorities by when the log committed them. A recovery rotation in the
  reference commits one key and drops every other phrase and guardian, but only from the events
  after it: one committed earlier is not retired against its holder by a rotation under one
  added later, since its holder parts the chain beneath the addition, where the later one counts
  for nothing; RFC 0003's line for `identity rotate` is corrected to say so. And a rotation
  written over an event that already commits another holder's authority shares that event with
  them, so both count; the rotation that ends that holder is written over the last event before
  their authority was added.
- Two rotations of the same standing are not settled by the rule: each drive and device keeps
  the chain it held, and they may differ. Two that both count are two holders of one authority,
  and a phrase two people hold has two owners, as the threat model says of it; or two
  authorities the last shared event both commits; or one person's two rotations, the first over
  an event that commits another holder's authority and the second beneath it to end that holder,
  where the second stands over the holder's chain and not over the person's first. Two that do
  not count are two authorities each added after the parting. A side never ends on a chain that
  another it was shown stands over, so where two sides differ it is between chains the rule
  leaves to their holders, and the order of arrival decides no more than that. The contest
  window an event carries could decide between two of the same standing, and is not made to
  here.
- Whoever holds a recovery authority can shut every device out of every drive at once, by a
  recovery rotation that names only their own. The reference acts on a recovery rotation as soon
  as it sees one, and the contest window an event carries is still not enforced. This is a power
  that holder did not have at a drive before: the threat model says a recovery unpairs no device
  at a node, and that paragraph changes when this is built. It is also what taking an identity
  back from a thief requires, and where the person's rotation counts and the thief's does not it
  takes it back whoever moved first.
- One whose recovery claim passes learns, before any rotation, that the drive keeps the
  identity, its identifier, one event's sequence number and digest, which bound the length of
  its log, roughly how many authorities it commits, with hashes over the others' keys by which a
  key already known can be recognised, and no device's key. Asked again it learns whether the
  log has moved, and no drive counts the asking. The landlord learns that a holder of the
  authority asked and when, the key of the device that asked, and, until #48, the address it
  asked from, and holds nothing that shows any of it to anyone else. A listener on the path can
  tell a claim's connection from another by the lengths of its frames, and learns nothing of
  whose it is. The claim's exchange asks one new thing of a responder: it keeps its ephemeral
  value until it has read the frame that answers its first, and no longer.
- What the claim does not settle. The device extends an event it cannot check until the drive
  sends the rest of the log. A drive names an earlier event than its latest where the rule gives
  one, and the rotation parts the chain there and stands over the ordinary events written since;
  the device cannot tell that from a drive that names the wrong event, an earlier one to drop
  what was written since or a later one where its rotation is left to the side that holds the
  log. A guardian's claim is answered with a sealed copy no guardian opens, since the copy is
  sealed under a phrase. And a device the log admits can replace the sealed copy a drive keeps.
- A landlord learns the whole history of the person's devices, each key and the order in which
  it was admitted and withdrawn, from the first. Before, it learned the devices introduced to
  it. Two landlords of one person hold the same log and the same device keys, and can tell they
  serve one person. That is true today, since a vault holds the identity's log from enrolment.
- A contact who holds the person's log, to follow their key by it, holds the roots alone: no
  device's key, and no salt, without which no key can be tried against a root, not one a reach
  note has shown it either. A root is 32 bytes whether it is one leaf or the top of many, so it
  shows no count, and no two roots are alike, since each has its own salt. What the copy does
  show of the devices is which events name them: that the inception and each recovery rotation
  name some, and how many device events lie between and where, each of them one change and not
  which way. The digests are the same at every holder of the log, as the identifier is. A key
  shown under a root by a path would show, by the path's length, roughly how many keys lie under
  that root. What a device gives a contact of the log, and when, is not decided here: the seam
  is a log for each holder, derived from the identity's, and it is the recovery copy's question
  as much as the handshake's.
- An event that names devices costs a salt and the hashes of its root, and a drive or a device
  keeps the salt for as long as it keeps the event: a whole copy that has lost its salt can no
  longer be checked, or given whole to another. No holder refuses an event for the device keys
  it names, since a holder of the roots alone could not: a key in a Montgomery form already
  admitted is not admitted, and the event stands.
- A device that has not been shown a withdrawal still answers the withdrawn device, and still
  dials it. The wait above is every holder's, not only a drive's.
- A key takes part in an exchange in its Montgomery form and signs in its Ed25519 form
  elsewhere, and a pair handle already uses the first form between two static keys. A recovery
  authority's key is now such a key: it signs the log's rotations and takes part in a claim's
  exchange. The known analyses of one key in both uses hold while an exchange's output goes only
  into the key derivation and is never released, which is true of every use here.
- RFC 0002 called the handshake the signed form of the ceremony the browser's door runs. It is
  not that. The ceremony shows the initiator's key to whoever completes the ephemeral exchange,
  since neither side knows the other yet; here the initiator already holds the responder's key,
  so the first message is bound to it and neither key is shown to anyone else. RFC 0002's
  sentences are corrected. The browser's door keeps its ceremony, which admits a key a node has
  not met by a person's comparison, and pairing keeps its ceremony and its own version in its
  own first frame, which this RFC does not change, but for the certificate its socket presents.
- On the QUIC path two key exchanges run, QUIC's beneath the protocol's, the cost RFC 0002
  accepted.
- Left open, each a follow-up and not a reason to wait: the path to a vault, which is #48;
  whether the contest window an event carries should decide between two rotations of the same
  standing; whether a drive should name, and `identity rotate` write over, the last event before
  any authority was added after the one the rotation is under, at the cost of the events since,
  so that a device that holds only a phrase is not handed an event where another holder's
  rotation counts beside its own, which is #55; how guardians make a claim together, and what it
  opens for them; what keeps a device the log admits from replacing the sealed copy; the bytes
  of the payload, of a claim and its answer, of the frames that follow and of the log's events,
  which the golden encodings fix; what a drive keeps of what a device wrote while an event that
  admitted it stood and was then replaced; how a contest window bears on a recovery rotation's
  reset of the devices; a device whose admission lies more than 64 events past a drive's log and
  has no other device to go first; how a drive's own key is replaced; rekeying a channel that
  stays up; padding and timing, which the threat model names as not covered; whether a responder
  should remember first messages so a recording tells nothing; a witness or a standing that
  expires, should the wait above prove too long; who besides the first device may sign for the
  identity; how the pairing of a gateway or a co-signer, and its end, reach the person's other
  devices; a version for the pairing payload, which #12 asked of this page and which is
  pairing's to state; and a log for each holder, with what a device gives a contact of the log,
  whether a key is ever shown to a holder under a root, and whether a root should hide the count
  from one who is shown a path.

## Reaches

- `kotton-protocol`: the type crates gain the handshake's messages, the recovery claim and its
  answer, and the log's device event; the tenant-device types of #20 are not moved in, since the
  message they carry ends. In this pull request: RFC 0002's paragraph on the handshake and RFC
  0003's line for `device rm` are corrected as above; the threat model's row for a vault's
  landlord is corrected for what is already true, that it holds the identity's log, the keys of
  the devices it was told of, and the address a device dials it from; and `rfcs/README.md` asks
  of an RFC what #12 asked of this one. The threat model's rows for a landlord, for a person
  holding the phrase and for someone with your device, and the processes page's transports,
  change when the system does. By amendment, for #49, #50 and #53: RFC 0003's line for `identity
  rotate`, RFC 0002's sentence on what a drive accepts and RFC 0002's Reaches line for `kotton`
  are corrected. It ships in the minor version that names this RFC.
- `kotton`: the handshake over QUIC and over a circuit, in place of mutual TLS as the bearer of
  identity, answered in the user daemon with `kquicd` carrying its frames; a vault's operator
  admitted by the vault's operator set, a gateway and a co-signer by pairing and named in no
  log, a contact's door under the pair's keys for the hour, and pairing's socket under a
  certificate made for the connection; a device key for each person on a device; the log's
  device event, the inception and the recovery rotations naming keys under a device root their
  digests cover, with the salt and the keys beside it in a whole copy, which is checked against
  the root, and the root alone in a copy that verifies without them; the rule for two chains,
  with a recovery rotation's digest covering its authority and an event that sets a commitment
  refused unless the keys it carries make it; the first frame's wait, and the kind byte of the
  frame that answers the ask; the recovery claim as an exchange and a tag, in place of the
  signature a certificate carried, checked against the event the door answers from and answered
  with that event's name, a path and the sealed copy in place of the whole log; admission by the
  log at a vault and at a device, with the row for a tenant's device and the message that
  introduced one removed. Nothing is released, so no identity or log is carried over: the
  inception's digest changes with what it names. The reference's issue is opened when this
  lands, and #64 closes under it.
- `kotton-vault`: it admits a device by the log and its operator by its own set, and accepts the
  handshake over whatever it can listen on.
- `kotton-cli`: `device rm` says what the wait is and when to rotate, `identity rotate` says
  when its rotation will not stand, and `status` shows a drive whose log is behind.
- `kotton-relay`, `kotton-gateway`, `kotton-portal`, `kotton-dist`: nothing.
