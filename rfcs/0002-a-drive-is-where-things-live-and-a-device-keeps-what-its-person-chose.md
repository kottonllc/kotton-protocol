# 0002: A drive is where things live, and a device keeps what its person chose

## Context

Replication is by location today. A named root has members, the devices that hold its key, and
targets, the places it is kept; a person adds a vault as a target with `sync add`, and the
engine runs one worker per target. A mount records pins and a cache budget that nothing yet
enforces. So the pieces of a drive exist, some as records. What is not decided is the model
they serve: members replicate to one another as well as to targets, the bare `sync` verb fans
out to every node the person has paired, the first-release page says a second device "brings
the other's things", and enrolling a vault asks "keep your things here too?". A person reading
those sees a mesh that copies everything everywhere, with the vault as one more node.

Identity rides in the transport. Devices speak QUIC with mutual TLS, each certificate derived
from the device's identity-bound key, and a node accepts connections only from the devices it
has paired with. That is one handshake for QUIC. Pairing itself is a committed ephemeral key
exchange a person confirms by comparing six digits, which introduces a device's key to the set;
the browser's door signs that exchange's transcript under a device key because a browser has no
QUIC. So a device proves itself one way over QUIC and another way where QUIC is absent.

What, then, is the storage model: where things live, what a device holds, how a device proves
itself to a drive, and how it reaches one?

## Decision

**A drive is where a person's things live.** A drive is a tenancy on a vault: the vault keeps
sealed packs it cannot open, for the tenants it admits, and a person's drive is their tenancy
on one, whether they run the vault at home or rent it. A person may have several drives, on one
vault or many; anything may be kept in one. A device names a drive by the vault's key and the
tenancy, wherever the vault runs.

**A device keeps what its person chose.** For each drive and each folder, the person says
whether this device keeps a local copy or fetches on demand; the mount's pins and cache budget
are that choice. Sync is a device keeping its chosen folders current with the drive: it sends
its changes to the drive and takes the drive's changes for the folders it keeps. A device is
not a target of another device; devices replicate to drives, and read from them. The bare
`sync` verb syncs this device's drives, not every node.

**Each side proves itself under its key, in the protocol.** Pairing introduces a device's key
to a person's set, as it does now. After that, every connection a device opens to a drive or to
another device begins with a handshake under both keys, defined by the protocol and run the
same way over every transport: each side signs for itself, a device for itself and a drive or
the other device for itself, so nothing standing between them, a relay included, can answer as
either. It is the signed form that the browser's door already runs of the exchange pairing runs
unsigned: an ephemeral exchange keys the channel first, and the keys and the signatures over
the transcript cross only inside it, so the path sees ephemeral values, authentication tags and
ciphertext and never a durable key. The transport need add no confidentiality of its own, and a
relay or a nearby listener learns no more of who is talking than it does now. A drive accepts a
connection only from a key in the set it admits, and a key removed from the set is refused at
its next connection. A transport carries bytes and confers no identity: QUIC on a local network
or across the internet, a circuit through a relay, and a link such as Bluetooth are transports,
and a drive is reached over any of them the same way. The QUIC path that exists is the first
transport, and until the handshake exists it keeps its mutual TLS.

**The relay joins what cannot reach each other directly.** It carries circuits and holds
nothing past delivery, as RFC 0001's row says; in this model the circuit a device needs most is
the one to its drive.

**Devices know each other, and a person may hand one device's things to another directly.**
Pairing and admission stay, and a device recognises the person's other devices. Recognition is
the identity layer, not replication. An act between two devices with no drive between them is a
verb the person invokes, one at a time, over the same transports. The first such verb is a
transfer. On one device a person names something that device holds, a file or folder from its
local tree or from a folder it keeps, and one of their other devices. The two must reach each
other directly, on one network; a transfer is refused when only a relay joins them, and a link
such as Bluetooth joins them when a transport for it exists. The receiving device asks its
person: where it has a compositor, the compositor draws the prompt, as it draws every prompt an
app must never draw; where it has only a terminal session, the daemon asks there; a device with
nobody to answer refuses when the offer times out. The person says where the bytes land. They
cross under whatever proves identity on that transport at the time, mutual TLS on the QUIC path
today and the protocol's handshake once it exists, and the two devices are in no standing
relationship afterwards: the receiving device is not a target of the sending one. What landed
is the receiver's, and if it landed in a folder the receiver keeps on a drive, the receiver's
own sync carries it there as it carries any change. The protocol defines the offer, the
acceptance, the refusal and the stream. It is built on the QUIC path that exists, right after
the model's edges above.

**Exit.** A person's things are on their drives, and a hosted drive is held by a landlord who
can decline to serve. Principle XIII is kept by one means the person alone controls: a device
that keeps every folder of a drive holds the whole of it, and needs nobody. So the device that
creates a drive keeps it whole until its person says otherwise; a device paired later keeps
nothing until told; and whenever no device of the person's keeps a drive whole, the person is
told so, where they choose what each device keeps. Two more things help and are not exits: a
drive may be exported whole, sealed, to another drive or a disk, which needs the landlord to be
serving; and a person may have several drives, which are as full as something filled them. The
first release has the default and the telling; the export is the drive's verbs' when the vault
is cut.

## Consequences

- One path to design and hold: device to drive. Member-to-member replication and the bare
  `sync` verb's fan-out leave the engine.
- The person's choices are the policy. A device holds the folders it was told to keep; a device
  with no reachable drive has those and nothing else, as every drive a person has used behaves.
- A drive orders what it receives; devices that hold the key resolve what the order means. A
  drive that cannot read a pack cannot merge it. What a drive retains of older versions is the
  drive's verbs' question, when the vault is cut.
- A second device joins by being paired and told which folders to keep. The first-release
  page's line for a second device says that, and `enrol` stops asking whether to keep things on
  a vault: a vault is where things are kept.
- One handshake replaces one and a half. Mutual TLS stops carrying identity, so a drive may be
  reached over a transport that cannot do it, and a vault behind a TLS terminator is one
  implementation among any. On the QUIC path two key exchanges then run, QUIC's own beneath the
  protocol's; that cost is accepted for one identity story over every transport. "Nothing listens for strangers" becomes "nothing answers a stranger
  past the handshake", and the threat model says so when the system changes.
- The relay learns no more than it does now: a circuit's two ends, as handles.
- Two of a person's devices on different networks have no direct verb; what one is to hand the
  other goes by way of a drive.
- The direct transfer is the first nearby-device verb. Two devices on one network sharing a
  fetch, or working together with no drive reachable, are later verbs on the same layers, each
  decided by its own RFC; nothing here forecloses them.

## Reaches

- `kotton`: the sync engine (members are not targets), `kquicd` and the peer listener (the
  handshake in the protocol; mutual TLS kept until it exists), the drive's verbs in `kvaultd`
  (today a vault keeps one pool per tenant; several drives for one person on one vault is new
  work there), the pins and budget the daemon enforces, `enrol`, and the daemon's side of the
  transfer: the offer received, the prompt asked, the bytes placed.
- `kotton-cli`: the keep policy's verbs, the bare `sync`, and the transfer verb.
- `kotton-dist`: the nine lines' second-device line.
- `kotton-protocol`: the handshake, the drive's verbs, the keep policy's types, and the
  transfer's offer, acceptance, refusal and stream, and the question a daemon puts to a terminal
  session and its answer, shipping in the first tag, `v0.1.0`, which follows the golden-encoding
  check, or in a patch after it, since each is a new request.
- `kotton-compositor`: the transfer's prompt, where a device has one.
- `kotton-relay` and `kotton-vault` when cut.
- `design/processes.md` and `design/threat-model.md` here describe the system as it is and are
  corrected in the pull request that changes it.
