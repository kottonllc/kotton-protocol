# The threat model

What each party can and cannot learn or do, as the code stands. Where the answer is "today,
and not by design," it says so.

## A vault's landlord

Learns: that a tenant exists and under which operator key; the tenant's identity, as its key
log, which the vault holds from enrolment, and the keys of the tenant's devices it was told of;
how many trees it keeps for them, the size of each, and when one appears or leaves; when those
bytes change; and
which packs are read and written and when; and which of the tenant's devices is moving or making
a drive, and when. For a move it sees that already from which device writes. For a making, the
making lease is taken at one of the tenant's vaults whichever vault the drive is made at, so
that vault learns of a making it may otherwise never see: which device of the tenant is making a
drive, and when, with no name and no place. Cannot learn: a filename, a file's size, how many
files there are, which chunks belong together, or the shape of a drive's folders. Cannot be
compelled to produce plaintext, because it holds none and has no verb that returns any. A
warrant yields a byte count. Volume and access pattern are the known leaks; fixed allocation and
oblivious access are the known answers and are not built. By design (RFC 0007) a vault is
reached at the relay it stands at and learns no address a device connects from, but for a device
whose person set it to dial that vault directly. Today a device dials a vault's address before
it tries a relay's circuit, so the vault learns that address, until the reference's issue under
that RFC lands.

## A relay's operator

By design (RFC 0007) learns: handles, bytes and timing; which handles are awake; the address
each device and vault that stands at it connects from, and which of its admission keys made each
tag; for each circuit one of those devices asks it to carry to another relay, that relay's key
and address, when and how much, and the two handles of the open for their hour; and of a circuit
that arrives from another relay, that relay's address and no device's. Can tell that a node
which holds many doors at the end of many busy circuits is likely a vault, and, where it holds a
vault's line, which door is that vault's and so the vault's key. Cannot learn: whose a vault is,
who operates one, a tenant's name, which of the people who stand at it asked for a circuit
beyond the address they asked from, or any durable identity of a person on any route, because
every handle is per contact and per epoch. The reference's relay is built to hold nothing past
delivery, of what it carries outward as of what it carries in, and to write down no far relay,
no handle and no address; the run that would hold that of what it carries outward is not built,
a tenant cannot check it of a relay someone else runs, and no line here rests on it. Where a
person stands at several relays, each is handed the same. What a relay is handed of a person who
runs it is that person's own record. Of a tenant it is routing, at a relay the tenant chose and
leaves asking no one. A relay reached by a device whose person set it to dial directly learns
that device's address. Today no relay carries outward and a device connects itself to each
contact's relay, so that relay learns the device's address and the device's own relay learns
nothing of which relays it reaches, until the reference's issue under that RFC lands.

## The network a device is on

By design (RFC 0007) sees one remote address beyond itself for each relay the device stands at,
the same each day, with when and how much, and how many connections the device holds there, one
for each door it stands at, a number that follows its contacts; and on that connection a version
byte, fresh values, lengths and sealed bytes. Cannot learn which relays the person reaches, or
whom. A device whose person set it to dial directly shows this network each relay, or the vault,
it connects to. Today a device connects to each contact's relay and to its vault itself, and
this network sees each of them, until the reference's issue under that RFC lands.

## A listener between two relays

Sees the two relays' addresses, when and how much, and a version byte, fresh values, lengths and
sealed bytes: no handle, no key and no signature (RFC 0007). Where each relay carries one
person, it sees that those two people's relays speak.

## One who holds a vault's line and stands at its relay

By design (RFC 0007) can make the label of that vault's door for the hour and, where the relay's
rule lets them stand, hold a place at it. Learns when a first knock is made at that vault.
Holds, durably, the vault's key, which the line already gave. Joins that to the vault, and to no
person and no address. Is answered as nobody: what answers a knock is proved by the vault's key.

## One landlord running both

Learns what the two above learn, joined: which handles belong to the same person, when that
person's devices are awake, how often they write, and, where the relay is the one that person's
devices stand at (RFC 0007), which relays that person reaches and is reached from, and when. A
landlord who runs a vault and the relay that vault stands at, where the person's devices stand
elsewhere, learns the address of the person's relay and no device's. This is the cost of renting
both from one provider. It is shown to each of your contacts as a readout, and the way out is to
rent the vault elsewhere and move, which needs the first landlord to serve while the move runs
and nothing else; a device of yours that keeps the drive whole cannot feed the move instead
today, and should.

## An app

Reaches what it was granted and nothing else, and has no network. Can draw inside its own window
and nowhere else; cannot read the clipboard or draw a prompt. Two gaps today: the signed install
approval is not yet read by the runtime, so an app's declared default is its grant until that
lands; and an extension could invent a name on first run and keep it, which is a working join
key, but nothing in the daemon calls the extension machinery yet, so no user is exposed to it.
By design (RFC 0004) cannot wear another publisher's name: a schema is named by the key that
publishes it, and a grant prompt shows the name the person gave that key or says they gave none,
so an app can ask for a stranger's schema but cannot make it look like a friend's. Today a
schema's name is a string anyone may declare and the prompt shows it as declared, until the
reference's issue under that RFC lands.

## A contact

Holds the key to what you shared with them and nothing else, and keeps what they were once sent.
One who operates the relay they stand at sees which relays you reach them from, every relay you
stand at, together (RFC 0007), which the reach note on their card already names, and never where
your device is, unless you set that device to dial directly. Can prove delivery only under a
device key your own reach note lists, so neither a vault nor a relay can forge a receipt. A
member removed from a channel cannot open the next epoch.

## A remote caller

Reaches a gateway that holds no user key and no persistent state, and from there only the
verbs its trust tier allows; no remote caller reaches a local-tier verb. A browser is trusted
as far as its vendor and its extensions are, and the Kotton client running in it is not: the
browser is the concession for a person who has not installed Kotton.

## Someone with your device

Locked: holds cached ciphertext, mutable pointers, and the session token file, which is on
disk in plaintext today and should not be. Unlocked: is you, until you rotate the keys that
device held, which the identity log records and your contacts follow.

## Someone with your phrase

Holds the identity, not only the data, because the phrase is the identity's authority. With the
phrase and the vault's line, which is not secret, they can run `id recover` on a device of their
own: it puts back the data roots from the copy the phrase opens, and rotates the identity onto
that device's key by an event the phrase's reserve key signs. Each node the device names checks
the event against the log, admits the device as yours, and syncs your things to it. It then
signs as you, and a contact who gets a note from it follows the log to its key and from then
refuses what your other devices sign with the key it replaced.

Your other devices are not told. The event goes to the nodes the recovering device names and to
no device, so each of yours keeps the log as it was and shows nothing; your nodes still admit
them, and they read and write your files as before, because the data roots did not change.
Messages are different: a contact who followed the log seals to the new key from then on, so
what they send reaches the holder's device and not yours, which open mail with the key the
rotation replaced; the holder receives as you as well as signing as you. Nothing you can do
withdraws the phrase that was used, so it stays the authority for whoever holds it: a recovery
you run on a new device takes the identity back onto that device's key, they can do the same on
another, and neither recovery unpairs the other's devices at a node. The rotation is in the log,
signed, for any node or contact it reaches; nothing yet shows it to you, refuses a device it
rotated away from, or lets you contest it, and the contest window the event carries is not
enforced. So guard the phrase as the identity itself, not as a password: keep it in a drawer.
Pairing never carries the seed or an unwrapped key on the wire, so a recording of a ceremony
does not yield it.

## The inference engine

Parses an untrusted model file inside a large runtime, so it is a separate process that can
open neither the store nor the network, and the daemon feeds it through a channel it
controls.

## This project

Has no exemption from any of the above. Its apps are confined like anyone's, its relay reads
nothing, its vault opens nothing, and its licence forbids a fork from removing a user's
ownership without saying so in source. There is no account with us, and nothing of yours on a
server of ours.

## What this model does not cover

Anonymity: the relay you stand at sees your source address and which relays you reach, and a
relay you reach sees your relay's address; where your relay carries you alone, that address is
yours at every relay you reach (RFC 0007). Nothing here hides who is talking to a relay, only
what they say and to whom. Kotton runs over the network you choose and adds no hop to it; a
tunnel or an onion route beneath is yours to add. Traffic analysis across relays, and padding
and timing on any leg. A compromised build of Kotton itself, which is what reproducible builds
and signed releases are for and which are only partly in place. A release's signature proves its
sums were signed by a key in `infra/release/allowed_signers`, read from the repository at the
release's tag, so a first install trusts the host that serves the repository for that key
([decision 0007](decisions/0007-the-release-key-is-read-from-the-repository.md)). A post-quantum
adversary: the ciphers are modern and not post-quantum, and this model takes no position on it.
