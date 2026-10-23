# 0002: A drive is where things live, and a device keeps what its person chose

## Context

Replication is by location today. A named root has members, the devices that hold its key, and
targets, the places it is kept; a person adds a vault as a target with `sync add`, and the
engine runs one worker per target. A mount may pin paths and cap its cache. So the pieces of a
drive exist. What is not decided is the model they serve: members replicate to one another as
well as to targets, the bare `sync` verb fans out to every node the person has paired, the
first-release page says a second device "brings the other's things", and enrolling a vault
asks "keep your things here too?". A person reading those sees a mesh that copies everything
everywhere, with the vault as one more node.

Identity rides in the transport. Devices speak QUIC with mutual TLS, each certificate derived
from the device's identity-bound key, and a node accepts connections only from the devices it
has paired with. That is one handshake for QUIC. Pairing itself is a committed ephemeral key
exchange a person confirms by comparing six digits, which introduces a device's key to the
set; the browser's door signs that exchange's transcript under a device key because a browser
has no QUIC. So a device proves itself one way over QUIC and another way where QUIC is absent.

What, then, is the storage model: where things live, what a device holds, how a device proves
itself to a drive, and how it reaches one?

## Decision

**A drive is where a person's things live.** A drive is a tenancy on a vault: the vault keeps
sealed packs it cannot open, for the tenants it admits, and a person's drive is their tenancy
on one, whether they run the vault at home or rent it. A person may have several drives, on
one vault or many; anything may be kept in one. A device names a drive by the vault's key and
the tenancy, wherever the vault runs.

**A device keeps what its person chose.** For each drive and each folder, the person says whether
this device keeps a local copy or fetches on demand; the mount's pins and cache budget are that
choice. Sync is a device keeping its chosen folders current with the drive: it sends its
changes to the drive and takes the drive's changes for the folders it keeps. A device is not a
target of another device; devices replicate to drives, and read from them. The bare `sync` verb
syncs this device's drives, not every node.

**A device proves itself to a drive under its device key, in the protocol.** Pairing introduces
a device's key to a person's set, as it does now. After that, every connection a device opens
to a drive or to another device begins with a handshake under that key, defined by the protocol
and run the same way over every transport; it is the signed form of the exchange the browser's
door already runs, and it keys the channel, so the transport need add no confidentiality of its
own. A drive accepts a connection only from a key in the set it admits, and a key removed from
the set is refused at its next connection. A transport carries bytes and confers no identity:
QUIC on a local network or across the internet, a circuit through a relay, and a link such as
Bluetooth are transports, and a drive is reached over any of them the same way. The QUIC path
that exists is the first transport, and until the handshake exists it keeps its mutual TLS.

**The relay joins what cannot reach each other directly.** It carries circuits and holds
nothing past delivery, as RFC 0001's row says; in this model the circuit a device needs most
is the one to its drive.

**Devices know each other, and a person may hand one device's things to another directly.**
Pairing and admission stay, and a device recognises the person's other devices. Recognition is
the identity layer, not replication. An act between two devices with no drive between them is a
verb the person invokes, one at a time, over the same transports. The first such verb is a
transfer: a person on one device names a file or folder and one of their other devices on the
same network or link; that device's person is asked, and says where it lands; the bytes cross
directly under the handshake, and nothing is replicated afterwards. It is built on the QUIC
path that exists, right after the model's edges above, and it is the shape every later nearby
verb takes.

**Exit.** A person's things are on their drives, and a hosted drive is held by a landlord who
can decline to serve. Principle XIII is kept by three means the person controls: a device may
keep every folder of a drive, so one device holds a full copy; a drive may be exported whole,
sealed, to another drive or a disk; and a person may have several drives. The first release
offers the first; the other two are the drive's verbs when the vault is cut.

## Consequences

- One path to design and hold: device to drive. Member-to-member replication and the bare
  `sync` verb's fan-out leave the engine.
- The person's choices are the policy. A device holds the folders it was told to keep; a device
  with no reachable drive has those and nothing else, as every drive a person has used behaves.
- A drive orders what it receives and holds every version it is given; devices that hold the
  key resolve what the order means. A drive that cannot read a pack cannot merge it.
- A second device joins by being paired and told which folders to keep. The first-release
  page's line for a second device says that, and `enrol` stops asking whether to keep things
  on a vault: a vault is where things are kept.
- One handshake replaces one and a half. Mutual TLS stops carrying identity, so a drive may
  be reached over a transport that cannot do it, and a vault behind a TLS terminator is one
  implementation among any. "Nothing listens for strangers" becomes "nothing answers a stranger
  past the handshake", and the threat model says so when the system changes.
- The relay learns no more than it does now: a circuit's two ends, as handles.
- The direct transfer is the first nearby-device verb. Two devices on one network sharing a
  fetch, or working together with no drive reachable, are later verbs on the same layers; nothing
  here forecloses them.

## Reaches

- `kotton`: the sync engine (members are not targets), `kquicd` and the peer listener (the
  handshake in the protocol; mutual TLS kept until it exists), the drive's verbs in `kvaultd`,
  `enrol`.
- `kotton-cli`: the keep policy's verbs, the bare `sync`, and the transfer verb.
- `kotton-dist`: the nine lines' second-device line.
- `kotton-protocol`: the handshake, the drive's verbs and the keep policy's types. No protocol
  version is named: nothing is tagged, so no shipped message changes meaning.
- `kotton-relay` and `kotton-vault` when cut.
- `design/processes.md` and `design/threat-model.md` here describe the system as it is and are
  corrected in the pull request that changes it.
