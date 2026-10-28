# The threat model

What each party can and cannot learn or do, as the code stands. Where the answer is "today,
and not by design," it says so.

## A vault's landlord

Learns: that a tenant exists and under which operator key; how many trees it keeps for them,
the size of each, and when one appears or leaves; when those bytes change; and which packs are
read and written and when. Cannot learn: a filename, a file's size, how many files there are,
which chunks belong together, or the shape of a drive's folders. Cannot be compelled to produce
plaintext, because it holds none and has no verb that returns any. A warrant yields a byte
count. Volume and access pattern are the known leaks; fixed allocation and oblivious access are
the known answers and are not built.

## A relay's operator

Learns: handles, bytes, timing and source addresses, and which handles are awake. Cannot
learn: which node is a vault, whose it is, who operates one, a tenant's name, or any durable
identity on any route, because every handle is per contact and per epoch. Holds nothing past
delivery.

## One landlord running both

Learns what the two above learn, joined: which handles belong to the same person, when that
person's devices are awake, and how often they write. This is the cost of renting both from
one provider. It is shown to each of your contacts as a readout, and the way out is to rent
the vault elsewhere and move, which needs nothing from the first landlord.

## An app

Reaches what it was granted and nothing else, and has no network. Can draw inside its own
window and nowhere else; cannot read the clipboard or draw a prompt. Two gaps today: the
signed install approval is not yet read by the runtime, so an app's declared default is its
grant until that lands; and an extension could invent a name on first run and keep it, which
is a working join key, but nothing in the daemon calls the extension machinery yet, so no user
is exposed to it.

## A contact

Holds the key to what you shared with them and nothing else, and keeps what they were once
sent. Can prove delivery only under a device key your own reach note lists, so neither a vault
nor a relay can forge a receipt. A member removed from a channel cannot open the next epoch.

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
that device's key by an event the phrase's reserve key signs. Each node the device names checks the
event against the log, admits the device as yours, and syncs your things to it. It then signs as
you, and a contact who gets a note from it follows the log to its key and from then refuses what
your other devices sign with the key it replaced.

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

Anonymity: a relay sees your source address, and nothing here hides who is talking to a relay,
only what they say and to whom. Traffic analysis across relays. A compromised build of
Kotton itself, which is what reproducible builds and signed releases are for and which are
only partly in place. A release's signature proves its sums were signed by a key in
`infra/release/allowed_signers`, read from the repository at the release's tag, so a first
install trusts the host that serves the repository for that key
([decision 0007](decisions/0007-the-release-key-is-read-from-the-repository.md)). A
post-quantum adversary: the ciphers are modern and not post-quantum, and this model takes no
position on it.
