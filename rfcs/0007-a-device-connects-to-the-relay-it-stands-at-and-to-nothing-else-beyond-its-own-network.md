# 0007: A device connects to the relay it stands at, and to nothing else beyond its own network

## Context

RFC 0005 decided one handshake for every connection and left the path a connection takes to this
page. RFC 0003's test of the release has two people make first contact at one relay, and says
first contact between people who stand at different relays is not settled. Both are one
question, which machines a device connects to, and this page answers it once.

The reference answers it in three ways.

- **To another person, the device connects to their relay.** Everything one person hands
  another, a message, a channel copy, a share, a grant, a reach note or an acknowledgement, is
  one request at the pair's door for the hour, at a relay the recipient's reach note names. The
  sender's device opens that circuit itself, and the door's holder, the recipient's device or
  their vault, comes to the same relay from its side. So the relay that carries a delivery is
  the recipient's, and it is shown the address of each device that delivers there.
- **To a stranger, both devices connect to one relay.** Pairing by a spoken code meets at a
  relay both devices stand at, so two people who stand at different relays have nowhere to meet
  until one is invited onto the other's.
- **To its own drive, the device connects to the vault.** It tries a direct connection, a hole
  punch and a TURN relay before a relay's circuit, and it publishes its address under a label
  the vault derives, so a vault is shown the address a device connects from.

In each, a machine the person did not choose is shown where their device is, at every
connection, and the person's contacts decide which machines those are. No relay opens a
connection of its own. An open at a relay presents the caller's label for the hour, a signature
under it over the time, and, where the relay's rule is narrowed, a tag over the label and the
time under a key the relay's operator hands the people they invite.

A relay tells a caller who it is by signing a value the caller chose, and that is the one thing
its key signs. The connection beneath is HTTP, plain or under TLS as the relay's address says,
and what a device believes of a relay is the key and not a certificate. A narrowed relay reads a
tag under one of two keys: its operator's devices hold one, and the people the operator invites
are handed the other.

What does a device connect to? What does a relay do for a device that stands at it, for whom,
and under what bounds? What does each party then hold, over months, and which of it is durable?

## Decision

**A device connects to the relay it stands at.** A device stands at one relay, the one its
person set, whether they rent it, run it, or were invited to it by whoever does. Of the machines
that carry or keep people's things, relays, vaults and other people's devices, a device opens a
connection to that relay and to no other beyond its own network. It reaches a door at another
relay, a contact's or the one its own vault stands at, by asking its relay to carry a circuit
there. The far relay is shown a relay's address and no device's. A device resolves no name but
its own relay's. Three things are outside this sentence and are not changed by it: how a
person's own devices reach each other, doors on a local network, and an operator's device
reaching the relay or vault it operates, at the `host` verbs. One thing sets it aside, the
person's own setting, below.

**A relay answers every ask under its key.** A caller, a device, a vault or a relay carrying for
one, makes each ask of a relay on a connection of its own, inside a channel the relay keys: the
Noise Protocol Framework's NX pattern (revision 34), `Noise_NX_25519_ChaChaPoly_SHA256`. The
caller is the initiator and has no static key. The relay's static key is the Montgomery form of
its Ed25519 key.

```text
caller                                      relay
0. -> the version, one byte
1. -> e
2.                                          <- e, ee, s, es
3. -> what is asked, one sealed frame: to stand, to come, an open, or to be carried
4.                                          <- the answer, one sealed frame
```

The version is 1, and the prologue is the ASCII bytes `kotton-relay-` followed by that byte.
Each message is one frame, its length first as two bytes, most significant first. The caller
compares the key the second message delivers with the Montgomery form of the key it holds for
that relay and closes if they differ; a caller that holds none, which is first contact, below,
takes the key delivered. Either side closes if an exchange yields all zeros. A relay waits 10
seconds, counted from when it accepts the connection, for the version byte and the first frame,
and 10 more for the ask, and closes on a connection that has not sent them, having written
nothing after its second message. A channel carries one ask and one answer. After an answer that
joins a circuit the channel ends, and the connection carries the circuit's bytes as they are,
sealed by nobody but the circuit's ends; after an answer that lets a holder stand the channel
stays, below; after reached, the channel stays for one open and its answer; after any other
answer the relay closes. The channel's hash, the 32 bytes Noise calls `h` once the second
message is read, covers both fresh values and the relay's key as that message sealed it, so no
two channels share one. The channel admits nobody and names nobody: it shows a caller that the
relay holds its key, and it gives an ask a value to be made over.

**A relay's key has one use, and a channel is not continued.** A relay is named, in its line and
in a reach note, by an Ed25519 key, a key by RFC 0006. Under this page that key does one thing:
its Montgomery form is the relay's static key in the channel. A relay signs nothing. A caller
that asks a relay who it is completes a channel and reads the key the second message delivered.
Every verb that names a relay proves it so, the `host` verbs and an operator's check of the
relay they operate among them. A channel has no ticket, no early data and no identifier of its
own, and nothing of one channel is presented on another: a device whose network changes, or
whose connection drops, makes a new channel and asks again. The channel runs over whatever
carries bytes to the relay's address, as RFC 0005's handshake does. Where that is TLS, on the
port and in the form of ordinary web traffic, so that a relay is reached from a network that
passes little else, the certificate is the operator's arrangement with that network; what a
device believes of a relay is the key the channel proves, whatever the connection beneath
presents.

- *Why.* A hop is sealed, and its far end proved, by a published handshake before anything is
  asked on it; that is the practice of TLS 1.3 and of what is built on it. This page takes the
  framework RFC 0005 took and not a certificate, because a relay is found by a key a person was
  told, and no authority and no name says whose it is. That framework's own rule is that a
  static key "should not be used outside of Noise", and a relay, unlike a device, has nothing
  else to sign, so its key is given one use and not two. Of data sent before a handshake
  completes, TLS 1.3 says it "is not forward secret" and that "there are no guarantees of
  non-replay between connections"; of a ticket, that its reuse "allows passive observers to
  correlate different connections". A channel has neither, so an open is good once and nothing
  joins two of a device's connections.
- *Read in.* *The Noise Protocol Framework*, revision 34, sections 7, 11.2 and 14, in the
  `noiseprotocol/noise_spec` repository. *The Transport Layer Security (TLS) Protocol Version
  1.3*, "0-RTT Data" and "Client Tracking Prevention", in the working group's own copy,
  `draft-ietf-tls-rfc8446bis` in `tlswg/tls13-spec`; the RFC as published could not be reached.
- *What it costs.* One exchange for each ask on each hop, a round trip that a continued
  connection would save. A person does not see it: a circuit, once joined, carries what it
  carries with no further exchange.
- *Not built.* The channel. The reference's relay tells a caller who it is by a signature, which
  ends when the reference holds this page; until then a relay's key is in both uses, under the
  analysis RFC 0005 names. A reservation and an open are each a request whose header carries
  the label, a signature over the label and the time, and the tag, on the connection as the
  relay's address gives it.

**A reservation is made in the channel, and a holder comes in one.** To stand at a door a holder
asks, in a channel, to stand: the ask names the door's label and whether the door is for
strangers, and carries the label's signature over the ASCII bytes `kotton-relay-stand`, the
channel's hash and the label, and, where the relay's rule is narrowed, a tag under a key that
lets its holder stand, HMAC-SHA256 over the ASCII bytes `kotton-relay-stand-tag`, the channel's
hash and the label. A reservation names one label. After the answer the channel stays for as
long as the holder stands, and on it the relay sends one sealed frame for each open it admits to
that door, naming the circuit by a value made for it: 32 bytes the relay draws fresh for that
circuit, sent to that holder and to no one else. The holder comes on a new connection, in a new
channel, whose one ask is to come and carries that value and nothing more, and the answer joins.
The ask to come is neither signed nor tagged: what admits it is the value, which crossed sealed
on the channel the holder stands on. A value is good for one coming, and ends when a holder
comes with it or when the 20 seconds a relay waits for a holder run out. So a reservation
carries no time, a relay reads no clock to admit one, and no label, signature or tag crosses
outside a channel on any leg.

**An open is good for one channel.** An open names the door's label and the caller's label, each
an Ed25519 public key, and carries a signature and, where the relay's rule is narrowed, a tag.
The signature is the caller's label's, over the ASCII bytes `kotton-relay-open`, the channel's
hash and the door's label. The tag is HMAC-SHA256 under an admission key, over the ASCII bytes
`kotton-relay-open-tag`, the channel's hash, the caller's label and the door's label. So what an
open presents is bound to one open, since a channel carries one ask; to one label, the door's;
and to one relay, since the hash covers that relay's key and its fresh value, and no other relay
can complete a channel with that hash. Nothing in it is good on another connection, so an open
carries no time and a relay reads no clock to admit one.

**A relay's rule tells one who stands from one who opens.** A relay's rule is one of three, as
it was: only its operator's devices, the people its operator invites, or anyone. To stand at a
relay is to hold reservations there and to ask it to carry. To open is to open a door someone
who stands there holds. A narrowed rule reads a tag under one of three keys. The operator's own
devices hold the first, and it lets them stand and open. A person the operator invites to the
relay is handed the second, which lets their devices stand and open, and with it the third, to
pass on. The third is the key a reach note hands a contact and a vault hands a tenant: it lets
its holder open and nothing else. Each key says one of those who hold it and never which. A
relay whose rule is anyone reads no tag, so it cannot tell one who stands from a stranger.

- *Why.* What a key lets its holder do is one thing. To stand is to be carried outward, and
  what a relay carries outward is seen as coming from the relay's address; to open is to reach a
  door someone who stands there holds. Practice keeps the two apart. A proxy that sends outward
  under its own address serves its own people only, since otherwise traffic is "attributed to"
  the proxy, and such a proxy "ought to restrict its use to authenticated users". At an onion
  service's introduction point, the cell that establishes the point is signed under the
  service's key, and the cell that introduces a client is admitted by what the client knows of
  the point.
- *Where this page leaves practice.* Practice gives each person a credential of their own, so
  that one can be withdrawn and each can be counted. That is a row for each person at a relay,
  and a history under it, which principles III and VII do not allow. Here a key says one of
  those who hold it and never which. What a person notices: an operator who removes one person
  rotates the key that person held and hands the new one to those who remain, and the key
  before it is read until the next rotation. The seam is a token that proves its holder is one
  of those admitted and cannot be linked to another, as Privacy Pass issues, "unlinkable proofs"
  in place of a credential that follows a person.
- *Read in.* *Proxying UDP in HTTP* (RFC 9298), Security Considerations, in the working group's
  own copy, `draft-ietf-masque-connect-udp` in `ietf-wg-masque/draft-ietf-masque-connect-udp`.
  *Tor Rendezvous Specification, Version 3*, sections 3.1.1 and 3.2, `rend-spec-v3.txt` in the
  `torproject/torspec` repository. *The Privacy Pass Architecture* (RFC 9576), Introduction, in
  the working group's own copy, `draft-ietf-privacypass-architecture` in
  `ietf-wg-privacypass/base-drafts`. The RFCs as published could not be reached.
- *Not built.* The third key, and a rule that reads which of the three made a tag. The
  reference's relay reads two, and a reach note hands the one that is handed to the invited.
  Unlinkable tokens are not designed.

**A device asks its relay to carry one open.** The ask names the far relay by its key and its
address, a host and a port, as a reach note or a vault's line gives them, and carries a tag
under a key that lets the device stand: HMAC-SHA256 over the ASCII bytes
`kotton-relay-carry-tag`, the hash of the channel between the device and its relay, the far
relay's key, or 32 zero bytes where the device holds none, and the address.

```text
device                     the relay it stands at            the far relay
1. channel, as above
2. -> to be carried: the far relay's
      key and address, and the tag
                           3. channel, as above, as the caller
4. <- reached: the key the far relay
      proved, and that channel's hash
5. -> an open, made over that hash
                           6. -> the open, as it was given
                           7.                                <- its answer
8. <- the far relay's answer
   then, if it joined, the circuit's bytes both ways, as they are
```

The carrying relay connects to the address, makes the channel as its caller, and closes unless
the key delivered is the key the device named; where the device named none it takes the key
delivered. It tells the device the key and that channel's hash. The device makes its open over
that hash, the carrying relay passes it on unchanged as that channel's one ask, and passes the
answer back. If the answer joins a circuit, the carrying relay carries the circuit's bytes
between the two connections until either closes, and then closes the other. The device does not
check what its relay tells it of the far relay, and need not: an open made over a hash is good
only at the machine that completed that channel, and what the circuit then carries is sealed to
the door's key, which no relay holds.

**What a relay carries outward, and what it cannot.** Each bound is stated with what makes it
necessary, and where a shape does the work no rule is added.

- *For whom.* For a caller whose tag lets it stand, and for no other. The tag is necessary
  because a relay must be reachable from any network and what it carries outward is seen as
  coming from its address. A relay whose rule is anyone carries outward for nobody, because it
  reads no tag and could not tell whom it carried for; that is the absence of the thing and not
  a check on it.
- *What.* One open, for each ask. What a relay passes to another relay is the one frame of an
  open, and after it joins, a circuit's bytes. It passes no ask to be carried, so no relay is
  asked by a relay to carry, and a path has two relays at most. It is not a tunnel: before the
  far machine has completed a relay's channel under the key named, the carrying relay sends it
  only bytes the carrying relay made, and nothing the device chose but the address.
- *To where.* To the address the device gave, once, and to nowhere the far relay then names. The
  proof of the key is necessary because the address is the device's word. A relay opens an
  outward connection only to an address that is reached from anywhere: one the IANA
  special-purpose address registries for IPv4 and IPv6 do not list as other than globally
  reachable, so not loopback, not link-local, not a private or shared range and not unspecified;
  and, as rules of this page's own, not a multicast address and not an address of the relay
  itself. Where the device gave a name, the bound is read on each address the relay would
  connect to. The bound is necessary because a relay stands on a network of its own as well as
  on the internet, and a carry is for the second: a relay carries between relays that anyone can
  reach, and a relay on an operator's own network is reached by the devices on that network. A
  relay's operator may name addresses outside the class that the relay carries to, which is how
  two relays on one machine or one network meet: in the relay's own configuration, on its
  machine, by address and port; no ask and no `host` verb sets it; and it is empty until named.
  To any other address the relay opens nothing and answers not reached. The registries are those
  RFC 6890 set up; that RFC could not be reached when this page was written and is cited by its
  name.
- *How many.* One outward connection for each ask, opened by that ask and closed with it, so
  none outlives the connection that asked. In all, as many at once as its operator set; this
  bound is necessary because a tag says one of those who stand and never which, so a relay has
  no smaller thing to count by.
- *For how long.* A carrying relay waits 10 seconds for the far relay's second message and 30
  for its answer to an open, and a relay waits 20 for a door's holder to come. The answer not
  reached is given at one time, 10 seconds after the ask, whatever happened at the address and
  whenever it happened, so that the answer is one in its timing as it is in its form. This page
  sets no limit on the life or the size of a joined circuit: a relay cannot tell a message from
  a drive's sync, and an operator who bounds what it carries bounds all of it alike.
- *What a refusal looks like.* An answer is one of: joined; nobody holds that door; not
  admitted, in one form whatever the caller lacked; full, which says to try again; and, from a
  carrying relay, reached, or not reached, in one form whether the far machine did not answer,
  did not complete the channel or proved another key. A carrying relay marks the far relay's
  answer as the far relay's. No answer carries a reason beyond its kind.

The values of the waits are written beside the golden encodings.

**The far relay admits the open as it admits any.** It reads the signature and, where its rule
is narrowed, the tag, on its own channel, and tells the door's holder to come, as it does for an
open a device makes. A device that opens a door at the relay it stands at makes the same ask on
the same route, so the far relay is shown nothing that says the caller is a relay, gives the
carrier no standing, keeps no list of relays, and is not asked to know. No relay proves a key of
its own to the relay it carries to, and no relay decides which relays it speaks with by a key:
that would put a durable, proved key on a route and need a list of relays, and a list of relays
is an asset in whoever holds it and an authority over who can meet whom.

**A door for strangers admits an open that carries no tag.** A holder may make a reservation for
strangers: the relay admits an open that names it without a tag, whatever the relay's rule.
There are two kinds. A ceremony's, first contact's or a vault's invite's, is under a label a
spoken code derives, admits one open and ends. A vault's door for the hour, below, admits any
number, within the reservation's own bound of places. To an open without a tag at any other
door, and at no door, a narrowed relay answers not admitted, so the answer does not say whether
a door stands.

- *Why.* A place to meet lets one who has not been met knock, and what stands behind the door
  proves itself and admits. That is the practice of a rendezvous: at an onion service's
  introduction point a client needs to know the point and its key and nothing more, and the
  service, not the point, decides whom it answers. Here the one who knocks must make the door's
  label, which takes the code or the vault's line, and what answers is the ceremony or RFC
  0005's handshake, which a relay cannot stand in for. Were a narrowed relay to ask a tag at
  such a door, a person recovering from a phrase would need a relay's key they have no device
  left to hold, and recovery would ask more of them than a phrase and a line. The relay's rule
  is not set aside by a stranger: the door is made by a holder the rule let stand.
- *Read in.* *Tor Rendezvous Specification, Version 3*, sections 3.1.1 and 3.2, as above.
- *Not built.* The door for strangers, of either kind. A label for a vault's door that only the
  vault can stand under is not designed. Practice has an answer, a key blinded for each period:
  "Any client that knows the hidden service's public identity key can derive these blinded
  signing keys for a given period", and only the service can sign under them. It is read in the
  same specification's Appendix A and in *Key Blinding for Signature Schemes*,
  `draft-irtf-cfrg-signature-key-blinding` in the `cfrg` organisation's repository of that name,
  which is a draft and no RFC. It would be a second way to make an Ed25519 key, and RFC 0006 is
  read against it before it is taken.

**What the ends prove is not changed by a second relay.** Read against RFC 0005, rule by rule.

- *A transport carries bytes, and a process that holds no key answers nothing.* Each relay is
  such a process for the door's key. After the answer that joins, neither frames, seals nor
  reads what it carries, so the frames the ends exchange are the frames one relay carries.
- *The initiator holds the responder's key before it begins.* At a contact's door that key is
  the pair's derivation for the hour, and at a vault's it is the vault's, from enrolment or the
  line. No relay supplies it, and the first message is bound to it, so neither relay can answer
  as the door.
- *One who holds the responder's key can tell that the responder answers there, and no more.* A
  door's label is the public form of the key it answers under, so each relay on the path is such
  a one. What it can tell, that the door answers, the answer to the open already told it. It is
  admitted to nothing: a contact's door admits the key the pair derives for the initiator and a
  vault's admits by the log, and no relay holds either.
- *No durable key crosses outside the channel.* The two labels are the public forms of the
  hour's keys and are shown to both relays in the open; neither outlasts its hour. No device's
  key, no identity and no log crosses to either relay, on either leg.
- *A key stolen later opens no recorded connection, and nothing is resumed.* A relay that
  records what it carries holds what a listener holds, and neither relay keeps anything a later
  connection could continue from.
- *The first frame's wait.* It is the door's holder's, counted from when it accepts the circuit,
  and the second relay's delay falls inside it.

The ceremony of pairing keeps its own frames and its own version, as RFC 0005 left them, and
they cross two relays as they cross one.

**What crosses in the clear.** Between a device or a vault and its relay, standing, coming,
opening or asking to be carried, and between the two relays, on the relay's channel: the version
byte, each side's fresh value, each frame's length, and sealed bytes with their tags. The
relay's key crosses sealed, to whoever completed the exchange and to no listener, and a relay
shows it to anyone who asks who it is. The labels, the signature, the tags, the far relay's key
and address and the answers cross sealed. Then the circuit's own bytes, which are RFC 0005's: a
version byte, two fresh values, lengths and sealed bytes. No durable key is among what crosses
in the clear on either leg, and no signature. This is the row of a relay's operator for what
each relay opens and reads, and the row of the network a device is on, and of a listener between
two relays, for the rest; the threat model gains the last two with this page. The transport
beneath shows what it shows to whoever reaches it, before any frame: that something at the
address accepts the route.

**A relay keeps nothing of what it carries outward.** Principle X's zero lifetime and "holds
nothing past delivery" are the outward leg's as they are the inward one's: a relay holds the
bytes of a circuit only while it passes them, and what it knows of a carried circuit, the far
relay, the labels, the address that asked, lives in the memory of the two connections and goes
when they close. Principle VII holds it to more than not buffering: a relay writes down no far
relay's key or address, no label, and no line that sets a far relay beside the address that
asked or the time it asked. What its configuration holds of addresses outside the class it opens
to, its operator wrote there and no ask did. It counts how many circuits it has carried outward
and keeps no count for each far relay. What holds this is the test that holds the inward leg,
run over a relay that carried outward: its state directory and everything it wrote, read whole
afterwards, hold no far relay's address or key and no label, but what its operator named in its
configuration. Principle VII stays labelled, held by review.

**A delivery goes to each relay the recipient's note names, through the sender's own.** For each
relay in the note, in the note's order, and then for the relay the device stands at: where it is
the relay the device stands at, the device opens the pair's door there; where it is another, the
device asks to be carried there, and makes its open with the tag under the key the note hands
for that relay. A channel copy goes the same way to each member. What a device says when a
delivery does not go, each by name: its own relay is not answering, or is not the relay it
knows; the far relay was not reached; the far relay refused the open; nobody holds the door
there; its own relay declined to carry, or is full. What is waiting stays in the outbox and is
tried again. A person whose relay is down sends nothing beyond their network until it returns or
they stand at another, which is `host relay set` and a reach note their devices sign again,
asking no one. Nothing falls back to a direct dial by itself.

**What a relay says of a far relay is its own word, in one form, and changes nothing a device
holds.** Everything a device is told of a far relay, but for a circuit that joined and then
proved itself by RFC 0005's handshake, it is told by the relay it stands at: that the far relay
was reached or was not, and the far relay's answer to an open. Not reached is one answer,
whether the far machine did not answer, did not complete the channel or proved another key. The
device says each as its relay's word, naming its relay as the one that says it. It acts on none
of them but to try again: the note stays as its signer left it, what waits stays in the outbox,
the next try is made as the first was, and no key, no relay and no path is added or dropped on
a relay's word. One thing is read against it: at first contact, where the device named no key,
the key its relay says was reached is compared, inside the ceremony, with the issuer's record,
and a difference ends the ceremony. The key the device then keeps for that relay is the one the
issuer's record names, and never the one its relay reported.

- *Why.* An intermediary that cannot reach the next hop says so, and the one it serves shows
  that to its person: HTTP has a status for an intermediary that "received an invalid response
  from an inbound server" and one for an intermediary that "did not receive a timely response
  from an upstream server", and the field that carries such a report is defined with the
  warning that "the field's content is not verified". This page takes that as it is: a report,
  believed for what to show and for nothing else. It does not take the detail practice allows.
  The same document asks for care in "what information to include". With one answer a relay
  tells whoever stands at it one thing of an address: whether a relay of the key named answered
  there.
- *What a person notices.* The message says their relay did not reach the far one, and it comes
  10 seconds after the try. It does not say whether the far relay is down or another machine
  now answers at its address. A device set
  to dial directly sees for itself and says which.
- *Read in.* *HTTP Semantics* (RFC 9110), sections 15.6.3 and 15.6.5, and *The Proxy-Status
  HTTP Response Header Field* (RFC 9209), sections 2 and 4, each as published, in the HTTP
  working group's own copy of its specifications, the `httpwg/httpwg.github.io` repository.
- *Not built.* Any of it: no relay carries outward, so no relay has a word to give.

**First contact is one case of this.** `contact pair` makes a code and waits at the relay its
device stands at, at a door for strangers under a label the code derives, and shows the code and
that relay's address for the person to say. `contact join <code> --relay <address>` asks the
joiner's own relay to carry one open to that address, naming no key, under a label made for that
dial; with no relay named it opens at the relay the joiner's device stands at. The named relay
admits the open by the door and is handed no key by the joiner, and the joiner is handed none by
it. Inside the circuit the ceremony runs as it does, and proves the two people to each other by
the code and the digits they compare; it is the ceremony and not the relay's key that the two
rely on, so a machine standing in at the address can deny the meeting and cannot join it. The
record each side sends names the relays that person stands at, by key, and a joiner whose relay
reached a key that is not among the issuer's ends the ceremony and says so. The named relay is
shown the joiner's relay's address, and the joiner's relay the named relay's, as each will be at
every delivery after.

A code and a relay's address are not a handle. The code is made for one ceremony and ends with
it or with its time; nothing is reached by it afterwards. A relay's address is durable, and
where the relay is one person's it stands for them, but it reaches nobody: a door is opened by a
label only the pair derives, and a relay tells a caller who holds no label nothing of who stands
there. What a contact keeps that is stable is the identity's key and the signed note that hangs
off it. Pairing by invitation onto one relay stays, for people who mean to share a relay.

**A drive kept on a host is reached at the relay its vault stands at, and by no other path.** A
vault stands at the relay its operator told it, and its line names that relay by key and
address. It holds a door there for each device its tenants' logs name, under the label the pair
derives for the hour, and a device opens that door, through its own relay where the relay is
another, and runs RFC 0005's handshake with the vault inside. A device takes no other path to a
vault: no direct connection, no hole punch and no TURN relay, and it publishes its address under
no label a vault derives. So a vault learns who and never where, and a relay learns where and
never who.

- *The first knock.* A device enrolling a vault by its invite meets it as first contact meets,
  at a door the invite's code derives, at the relay the invite names. A device the vault has not
  met otherwise, one recovering from a phrase or one admitted by an event the vault does not
  hold yet, opens the vault's door for the hour: a door for strangers under a label made from
  the vault's key, the relay's key and the hour, by HKDF-SHA256 with the vault's public key as
  its input, the relay's as its salt, and the ASCII bytes `kotton-vault-door` and the hour as
  eight bytes as its info, the output the seed of the label's key. Whoever holds the vault's
  line can make the label, and it tells them what the line already says, that this vault stands
  at this relay. The relay cannot make it unless it holds the line. What answers at the door is
  the vault, under its own key, by RFC 0005's handshake, which sends one who is not admitted
  nothing but the ask for its events.
- *Who names the relay.* The vault's operator names the relay the vault stands at, and the
  person names the relay their device stands at. A landlord who runs a vault and the relay it
  stands at is shown the address of the person's relay and no device's, so whether one party
  holds both who and where is decided by where the person stands and not by the landlord.
- *A vault the person runs themselves* is reached the same way, unless the person names it in
  the setting below.
- *What the relay then sees.* The relay a vault stands at carries every byte to and from it:
  volume and timing for each label for each hour. It can link a vault's hours by the one
  endpoint that holds them, and can tell that a node holding many doors at the end of many busy
  circuits is likely a vault. It cannot learn whose. A relay that holds the line knows which
  door is that vault's.
- *What it costs.* A reservation is a connection, and names one label. A vault holds a
  reservation for each device of each tenant for the hour and the one before, and one more for
  its own door; a relay's operator sizes the relay for the vaults it lets stand. A relay that is
  down takes the path to every vault that stands only there.

**A vault is infrastructure and not a device.** It connects to the relay it stands at, holds its
tenants' doors there and tries a sleeping sender's outbox there, as it does. It asks no relay to
carry, and it is not handed a recipient's relays: a vault knows whose it is, by the identity's
log, and a list of the relays its tenant's messages go to would set who beside whom at one
landlord. What waits for a recipient at another relay waits for one of the sender's devices to
wake.

**Dialling directly is the person's setting, for each device, and it is off until they say so.**
`device direct on` lets this device open a door at a relay it does not stand at by connecting to
that relay itself, with the same open; `device direct on <vault>` lets it connect to that one
vault at the vault's own address and to nothing else; `device direct off` ends either. With it
on, the device connects itself to every relay it does not stand at and asks its own to carry
nothing. The person is told what it shows before it is set: each relay this device reaches, or
that vault, sees the address the device connects from. `status` says it at every showing while
it is on. A device that stands at no relay reaches nobody beyond its network without it, and
says so, naming `host relay set` and the setting, when a person asks it to. With it on and no
relay, a device sends, and is reached only on a local network.

**What each party holds.** Over months, under this page, with the setting off.

| Party | Learns | Durable | Joins to |
|---|---|---|---|
| The relay a person's device stands at, rented or their own | the address the device connects from; the doors it holds, by label and hour; of what arrives, the address of the relay that carried it; of what the device sends, the far relay's key and address, when and how much, and the two labels of each open for their hour; which of its keys made each tag | the far relays' keys and addresses; the device's address for as long as its network keeps it | the address that asked, and so the person, at the grain of relays: which relays they reach and are reached from. An identity only where one landlord runs the vault too |
| A contact's relay | the address of the sender's relay, the two labels for their hour, when and how much; the address the door's holder connects from | the sender's relay's address | where that relay carries many, a crowd; where it carries one, that person, the same at every relay they reach. Nothing of where a device is |
| A contact's vault | the pair's key for the hour, and sealed bytes | nothing | nothing; it is shown no address of the sender or of the sender's relay |
| The person's own vault | who, by the log; when its tenant's devices are awake, and how much they write | the log | no address: it is shown no device's, and no relay's but the one it stands at |
| The relay a vault stands at | every byte to and from the vault, by label and hour; the address of each relay that carries to it | those relays' addresses; the vault's endpoint | a vault-shaped node to the relays that reach it; the vault's key, where it holds the vault's line; not to a person, unless one landlord runs the vault too |
| One who holds a vault's line and stands at the relay that vault stands at | when a first knock is made at that vault | the vault's key | that vault; no person and no address |
| One landlord running the relay a person stands at and their vault | all of the first and fourth rows, joined | the identity, and the relays it reaches | who the person is to which relays they reach and are reached from, and when |
| The network a device is on | one remote address, the relay's, always the same; when and how much, and how many connections the device holds there, one for each door it stands at, a number that follows its contacts | that address | nothing of whom the person reaches |
| A listener between two relays | the two relays' addresses, when and how much | both addresses | where each relay carries one person, that those two people's relays speak |

Each party holds something durable only as an address or a relay's key. The join key the design
removes is the device's address at every relay the person did not choose. The two it leaves are
the far relays at the person's own relay, and the person's relay's address at every far relay;
both are conceded, in Consequences.

**The threat model says of each party what this table says.** For each party that carries,
keeps or stands on the path of a person's things, the threat model states three things: what it
learns, which of that is durable, and what it joins to. It has a row for every party in the
table, the network a device is on and a listener between two relays among them, and it says
that a relay's operator can tell a vault by its shape. A row states what holds by this page,
and, until the reference holds it, what is true today, in the idiom the page already has for a
design not yet built.

- *Why.* A design that keeps who apart from what is stated by what each party sees, and by what
  it does not remove. "Privacy partitioning aims to separate *who* someone is from *what* they
  do", and of the timing and size of what a party carries, the same document says: "Although
  privacy partitioning does not obviate such attacks, it does increase the cost necessary to
  carry them out in practice." A row that left out a party on the path, or what a party can
  tell from shape and timing, would state the design as stronger than it is, and a person
  choosing where to stand would choose on it.
- *Read in.* *Partitioning as an Architecture for Privacy* (RFC 9614), "Privacy Partitioning"
  and "Limits of Privacy Partitioning", in the Internet Architecture Board's own copy,
  `draft-iab-privacy-partitioning` in `intarchboard/draft-obliviousness`; the RFC as published
  could not be reached.
- *Not built.* What the rows say by design. Each row's clause for today leaves when the
  reference holds this page.

**What a relay may refuse, and why the design does not settle into a few relays.** A relay can
tell apart, and so can refuse: a caller by which of its keys made the tag, and by the address it
connects from; a far relay, when it carries, by key and address; a carrying relay, when it is
the far one, by address; a door, by its label, for an hour. It cannot tell apart a person, a
pair beyond the hour, a message from a share from a drive's sync, or anything of what it
carries, so it can bound all of it and single out none. A relay need give no reason, and its
answer is one of the kinds above. A relay that declines to carry to another, or refuses what
comes from one, stops every person at it alike, each is told which relay refused, and each
leaves by standing at another and signing their note again.

What keeps this from settling into a few operators is what it does not have. No relay has
standing with another, so a relay stood up this morning is carried to and from as one that has
run for years, and no operator's judgment of another relay admits or refuses a person: admission
at a relay is the tag under a key the person there handed out. Nothing names a person by their
relay, so moving changes no contact's card but for a note the person signs. No relay is found
without being told, by a default or a directory. A relay is stood up from the lines `host mint`
prints, and the test of the release is two people who each run one. And a relay that serves
anyone carries outward for nobody, so there is no open carrier for small relays to be judged by.
One force pulls the other way and is stated: a relay one person runs alone is that person's
address at every relay they reach, so standing in a crowd is worth something. The answer this
page gives is a relay a few people share, run by one of them, and the seam is a second hop,
below.

## Consequences

- **What each party learns that it did not, and no longer does.** A contact's relay no longer
  learns the address of a device that delivers there; it learns the address of that device's
  relay. A vault's landlord no longer learns the address a device connects from. The relay a
  person's device stands at learns what it did not: each relay that device reaches, when and how
  much, and the labels of each open for their hour; and in place of the devices that deliver to
  its person, the relays that carry for them. A landlord who runs that relay and the person's
  vault learns which relays an identity reaches and is reached from. A relay's operator can tell
  a vault by its shape. The threat model's rows are corrected in this pull request.
- **Four concessions, each with its way back, on the concessions page in this pull request.** A
  rented relay learns which relays its tenant reaches. The ways back that ask no one: run the
  relay yourself; leave, by standing at another relay and signing the reach note again. The
  convenience, which asks someone: stand at a relay someone you trust runs. The seam: a second
  hop, so that no one relay sees both ends, and an open sealed to the far relay's key by the
  device, so that the carrying relay passes on a door it cannot read; neither is built. And a
  relay one person runs alone is that person's address at every relay they reach: one durable
  address, the relay's and no device's, and where the relay is kept at home, the home's. The way
  back is to stand at a relay others stand at too, which asks whoever runs it. Those admitted to
  a relay share a key, so one person is removed by a key rotated for all; the seam is a token of
  admission that cannot be linked, which is not designed. And a vault's door for the hour is a
  label whoever holds the vault's line can make, so a relay admits to it any such holder its
  rule lets stand; the seam is a label only the vault can stand under, a key blinded for each
  hour, which is not designed.
- **What forms about a person's traffic forms at the relay they chose.** They can leave it with
  nobody's cooperation, and a person who runs the relay holds it themselves: they can be asked
  for it, and nobody else can be asked instead. A person's ownership then turns on their own
  choices and not on their contacts'.
- **A relay is the address seen at the far end,** so its operator is the party asked about what
  came from it, and holds nothing to answer with but that it carried.
- **A person's relay being down stops them sending** as well as being reached, and a relay that
  declines to carry stops its tenant reaching a relay until they move. A rented relay gains that
  refusal; it gains no key, proves nothing to another relay and is believed by one for nothing.
- **What a device says of a far relay is its own relay's word.** Whether the far relay was
  reached, and what it answered, is told to the device by the relay it stands at and is shown as
  that. A person who doubts it stands at another relay, or sets the device to dial directly.
- **A person at a relay that serves anyone** reaches only doors at that relay, unless they set
  their device to dial directly.
- **A far relay that bounds by address bounds a whole relay at once.**
- **A relay's key is named in its Ed25519 form and used in its Montgomery form, and signs
  nothing.** A key and its negative share a Montgomery form, as RFC 0006 says, so a caller
  that compares forms takes either for the relay; only one who holds the relay's private key
  completes the channel under either.
- **What a person notices.** A send that crosses two relays makes one exchange more on each
  hop than a continued connection would, which is not seen. A drive kept on a host syncs
  through one relay or two and not along the shortest path between a device and its vault, so
  a large drive moves at the speed of the slowest relay on the way; the way back is the
  person's setting. A message that does not go says their relay did not reach the far one and
  not why. A person removed from a relay is removed by a key rotated for everyone there.
  Nothing else differs from a product that connects to its provider.
- **A second pattern, beside RFC 0005's.** The relay's channel is not the handshake that admits
  a device: it has no static key on the caller's side and admits nobody. It exists so that a
  relay's key is proved without a signature and what is asked of a relay crosses sealed, on a
  hop whose transport adds no confidentiality of its own.
- **A vault's door for the hour is a label whoever holds the vault's line can make,** and a
  relay admits to it any holder its rule lets stand. What answers a knock is proved by the
  vault's key and nothing else, so a knock not answered under that key is made again. A label
  only the vault can stand under is the seam, above.
- **A hosted vault that cannot hold a connection open to a relay** cannot stand at one, and is
  then reached only by a device its person set to dial it. What such a vault does instead is the
  adapter's RFC's to say.
- **RFC 0003 is amended in this pull request:** `contact pair` and `contact join`, `device
  direct`, what `host relay invite` hands, `status`, the test of the release, in which each
  person stays at their own relay, and the tag's conditions, which gain this page. RFC 0005's
  sentences on the path, RFC 0002's on which transport reaches a drive, and RFC 0001's row for
  the relay are corrected.
- **Left open,** each a follow-up and not a reason to wait: a second hop; an open sealed to the
  far relay; relays that prove themselves to each other, which this page sets aside for the
  reason given; how a person's own devices reach each other, and doors on a local network;
  reaching someone new without a spoken code; what a vault that holds doors learns, and removing
  a contact (#66); padding and timing across relays, which the threat model names as not
  covered; a label for a vault's door that only the vault can stand under; a vault standing at
  more than one relay; a reservation that carries many labels on one connection; whether the tag
  at an open guards anything the door's label does not; how a holder shows it still stands on a
  channel that is idle; a person's relay for each person on a machine two people share; the
  resolver a carrying relay asks for a far relay's name, a party that has no row yet; the bytes
  of the asks and answers, which the golden encodings fix; whether a relay says, inside a
  channel's answer, the address it announces, which the signed answer carried; a token of
  admission that cannot be linked, in place of a key shared by those admitted.

## Prior art

Each is read in its own published design, at the document named. Where a document could not be
reached when this page was written, the entry says so and asserts nothing.

- **SimpleX Chat's private message routing.** *Simplex Messaging Protocol*, version 20,
  "Proxying sender commands", and *SimpleX: messaging and application platform*, revision 4, in
  the `simplexmq` repository's `protocol/` directory. A sender asks a router it chose to connect
  to the router the recipient chose and to forward its commands, to "protect transport (IP
  address and session) anonymity of the sender from the router chosen (and, potentially,
  controlled) by the recipient". This page takes that shape whole: the forwarding party is the
  sender's choice, and the destination is shown it and not the sender. It differs in three ways.
  There the client seals its command to the destination router, so that the "proxy does not know
  how many different queues a connected client sends messages and commands to"; here the
  carrying relay is shown the two labels of an open for their hour, and sealing the open to the
  far relay is named as a seam. There the two routers add encryption between themselves under a
  secret agreed in a handshake, and the destination router answers with a certificate chain and
  a signed key; here the far relay's key is proved by an exchange and nothing is signed. And
  there the forwarded thing is a command to a queue; here it is one open, after which the two
  relays carry a circuit whose ends prove themselves to each other.
- **Signal's sealed sender.** Read in the reference implementation's own source and its
  comments, `rust/protocol/src/sealed_sender.rs` in the `signalapp/libsignal` repository; the
  published description it cites could not be reached, and nothing is asserted here of what the
  service keeps. The sender's public key is encrypted to the recipient, so the message as
  carried does not name its sender, and inside it travels a `SenderCertificate`, the sender's
  identifier, device and key under a signature whose signer is a `ServerCertificate`. This page
  takes the aim, a carrier that is not shown who sends. It differs in what stands behind it:
  there the sender is vouched for by a certificate a server signed, and here nothing vouches for
  a sender but the key the pair derives, and no server signs for anyone.
- **Oblivious HTTP (RFC 9458).** Read in the working group's own copy of the document,
  `draft-ietf-ohai-ohttp` in the `ietf-wg-ohai/oblivious-http` repository; the RFC as published
  could not be reached. A client sends a sealed request to a relay, which forwards it to a
  gateway. The relay "knows the origin and destination of an Encapsulated Request and Response,
  yet does not know the decrypted contents"; the gateway learns "only the Oblivious Relay
  Resource and the decrypted request"; and "the Oblivious Relay Resource cannot be operated by
  the same entity as the Oblivious Gateway Resource." That split, one party that knows who and
  one that knows what, held apart by who operates them, is the seam this page names as a second
  hop and does not build. This page's one carrying relay is the first of those two parties
  alone: it knows the origin and the destination. It takes one rule from the document as it
  stands, that a relay "MUST NOT add information when forwarding requests that might be used to
  identify Clients", which here is that a carrying relay passes an open on unchanged and adds
  nothing. It differs on one: there a relay "will only forward for Oblivious Gateway Resources
  that it has explicitly configured and allowed", which is a list, and here a relay carries to
  any machine that proves a relay's key.
- **Two proxies, neither of which sees both ends (RFC 9614).** Read as under "The threat model
  says of each party what this table says", above. The document describes a client that reaches
  a second proxy through a first, where the first "sees and proxies TLS data between the client
  and Proxy B" and "does not directly learn information that Proxy B sees", and states that
  "non-collusion is a fundamental requirement for privacy partitioning". That is the second hop
  this page names as its seam.
- **Magic Wormhole.** *Welcome* in the `magic-wormhole` repository's `docs/`, and *Mailbox
  Server Protocol* in `magic-wormhole-protocols`. Two people meet by a short code one says to
  the other: codes "are single-use and do not need to be memorized", a key exchange keyed by the
  code means an attacker, "e.g. a malicious Mailbox server", gets "only a single guess", and the
  server is "a simple WebSocket-based relay that delivers messages from one client to another".
  This page's first contact is that meeting: a code said aloud, a label made from it, and a
  relay that can deny and cannot join. It differs in where they meet. There "The URL of a public
  server is baked into the library for use as a default" and "Both clients must use the same
  mailbox server"; here there is no default, the issuer says the relay beside the code, and the
  joiner reaches it through their own. And there the number in a code is a nameplate, which the
  server's `allocate` command hands out; here the relay allocates nothing and is shown a label
  it cannot read.
- **A caution: what federated servers learn in Matrix.** *Server-Server API* in the
  `matrix-org/matrix-spec` repository. Persistent events "are broadcast from one homeserver to
  any others that have joined the same room", are "persisted in long-term storage and record the
  history of messages and state for a room", and are "signed using the originating server's
  private key"; every request between servers is "authenticated using public key digital
  signatures", naming "the server name of the sending server". That is the design this page
  declines under the name of relays that prove themselves to each other: servers with durable
  names and keys on every route, holding history. Here a relay holds nothing past delivery,
  proves its key to a caller and never names itself to another relay, and is shown labels that
  last an hour.
- **A caution: how email concentrated.** *Centralization, Decentralization, and Internet
  Standards*, read as the Internet-Draft `draft-nottingham-avoiding-internet-centralization` in
  its author's repository. Of SMTP: "Running one's own MTA has become considerably more onerous
  over the years, due in part to the increasingly complex mechanisms introduced to fight
  unwanted commercial e-mail", and "if a large MTA decides that e-mail from a small one is
  unwanted, there is significant impact on its ability to function, and little recourse"; and in
  general, "federation alone is insufficient to prevent or mitigate centralization of a
  function". This page is written against both mechanisms. Admission at a relay is a key the
  person there handed out and not the far relay's view of the sending one, and a relay carries
  outward only for those who stand at it, so the unwanted traffic that made judging senders
  necessary has no open carrier. It does not claim the outcome: the same document says that
  federation "can create the conditions necessary for a function to be decentralized, it does
  not guarantee that outcome", and the pull toward a crowd is stated above.

## Reaches

- `kotton-protocol`: the request, as types and golden encodings: the relay's channel, an open,
  the ask to be carried, a reservation and the ask to come, their answers, and the waits. In
  this pull request: the threat model's rows for a vault's landlord, a relay's operator, one
  landlord running both, a contact and what the model does not cover, with rows for the network
  a device is on and a listener between two relays; `design/concessions.md`, with the four rows;
  RFC 0003, RFC 0005, RFC 0002 and RFC 0001 as Consequences says. RFC 0006 is not changed: a
  relay's key, a label and the signature of an open are a key and a signature by its rule. The
  processes page's row for the relay changes when the system does. It ships in the minor version
  that names this RFC.
- `kotton-relay`: the relay's channel on one route for every ask, a reservation and a holder's
  coming among them, and a key that signs nothing; the class of address it opens to, and the one
  time of its answer not reached; an open good for one channel; a rule that tells one who stands
  from one who opens; a door for strangers; the first connection a relay opens, one open carried
  for one who stands at it, with its bounds and its answers; nothing written of what it carries
  outward. Done when it includes a run with two relays, each named in the other's configuration
  where they share a machine or a network, in which the far relay records the carrying relay's
  address and no device's.
- `kotton`: a device's delivery, first contact and path to a vault through the relay it stands
  at; each failure said by name, a far relay's as its own relay's word; a vault's door for the
  hour, and no address published for a vault; the setting to dial directly; the reach note
  handing the key that opens. Done when it includes the same run, in which the device holds one
  remote address, and the test of the release as amended.
- `kotton-vault`: its door for the hour; its line names the relay it stands at; it asks no relay
  to carry.
- `kotton-cli`: `contact pair` shows the relay beside the code, `contact join --relay`, `device
  direct`, and `status` says the setting and names each failure.
- `kotton-dist`: the test of the release, with each person at their own relay.
- `kotton-gateway`, `kotton-portal`: nothing.
