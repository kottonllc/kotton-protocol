# 0002: A drive is where things live, and a device keeps what its person chose

## Context

A person's things live in one place and are reached from every device they own. That is the
shape every drive a person already knows has: a place that holds everything, and devices that
hold what they were told to keep and fetch the rest when asked. Devices do not talk to each
other to stay current; each talks to the drive. One path carries everything, conflicts are
settled in one place, and "what is on this device" is a choice the person made, folder by
folder.

Kotton's drive is the vault: it keeps sealed packs it cannot open, a person runs one at home or
rents one, and a device trusts it by its key. The sync engine today treats the vault as one
peer among the person's devices and replicates a change to every peer it knows. Devices prove
who they are to each other inside the transport, by a certificate derived from the device key
in QUIC's mutual TLS, so identity exists only on transports that carry that handshake; the
browser's channel and pairing prove identity a second way, with their own handshake.

What, then, is the storage model: where things live, what a device holds, how a device proves
itself, and how it reaches a drive?

## Decision

**A drive is where a person's things live.** A drive is a vault the person created; a person may
have several; anything may be kept in one. A drive holds sealed packs and cannot read them. A
drive is addressed by its key, wherever it runs.

**A device keeps what its person chose.** For each drive and each folder a person says whether
this device keeps a local copy or fetches on demand. A device holds nothing from a drive it was
not told to keep. Keeping a folder current with its drive is the whole of sync: a device sends
its changes to the drive and takes the drive's changes for the folders it keeps. Devices do not
replicate to each other.

**Identity is proved in the protocol, over any transport.** A device proves itself to a drive,
and to another device, by a handshake under its device key that the protocol defines, the same
handshake pairing and the browser's channel use today. A transport carries bytes and may add
confidentiality; it confers no identity. QUIC on a local network or across the internet, a
circuit through a relay, and a link such as Bluetooth are transports, and a drive is reached
over any of them the same way. The QUIC path that exists is the first transport.

**The relay joins a device to a drive it cannot reach directly.** It carries a circuit and holds
nothing past delivery, as it does now. That is its one job in this model.

**Devices know each other.** Pairing and the admission rules stay: a drive admits the devices
the person paired, and a device recognises the person's other devices. Recognition is the
identity layer; it is not replication. Acts between two devices with no drive between them, a
transfer of bytes on one network or a link, are verbs the person invokes, one at a time, over
the same transports, and none is built by this RFC.

## Consequences

- One path to design and hold: device to drive. The engine's fan-out to every peer leaves, and
  with it the state that tracked every peer's position.
- The person's choices are the policy. The default is the drive's: nothing is on a device until
  the person says so, and a device with no reachable drive has what it chose to keep.
- Conflicts are settled at the drive, once, as every drive a person has used settles them.
- A second device joins by being paired and pointed at the person's drive. The nine lines'
  "add a device, then sync brings the other's things" becomes "add a device, choose what it
  keeps".
- The one handshake replaces two. Mutual TLS no longer carries identity, so a vault may be
  reached over a transport that cannot do it, and a drive run on infrastructure that terminates
  TLS for you is one implementation among any.
- Messages, like everything else, live in the drive and are read from every device there.
- Nearby-device features, a transfer without a relay, two devices on one network sharing a
  fetch, working together with no drive reachable, are later verbs on these layers. Nothing
  here forecloses them, and nothing here builds them.

## Reaches

- `kotton`: the sync engine, the vault's verbs, `kquicd` and `ksyncd`; the protocol-level
  handshake replacing mutual TLS as the carrier of identity; the per-folder keep policy and the
  CLI verbs that set it.
- `kotton-protocol`: the handshake, the drive's verbs and the keep policy's types, as one minor
  version, since the meaning of a sync message changes.
- `kotton-relay`, `kotton-vault` when cut.
- `design/processes.md` and `design/threat-model.md` describe the system as it is, and today
  it replicates to peers over mutual TLS; they are corrected when the system changes, in the
  pull request that changes it, not here.
