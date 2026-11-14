# 0003: The first release is a platform a stranger can use from the command line

## Context

RFC 0001 says what the product is made of and RFC 0002 says where a person's things live.
Neither says what `v0.1.0` is: which things a person can do with it, from what surface, on what
machines, and what it does not yet do. The versioning rule reserves `v0.1.0` for the
distribution a person installs, after the protocol's golden-encoding check exists; snapshots
(`v0.1.0-dev.N`) prove the pipeline before that and promise nothing.

The reference holds most of a platform already: identities and devices paired by six digits, a
sealed store with versions kept per device, a vault a person enrols and keeps folders on,
contacts and messages, typed records under schemas, grants, a host a person mints, recovery
from words that are stored nowhere. It holds them under the command names that grew with them:
some verbs at the top of the tree and some only inside the REPL, the store reached by its own
words (`keep`, `host:`) rather than the words a person already has for a filesystem, and
several nouns (`peer`, `config`, `net`, `grant`, `records`) for what a person would call one
thing.

What, then, is `v0.1.0`: what a person can do, how the verbs are named, where it runs, what
leaves the sealed tree and when, and what is deliberately not in it?

## Decision

**`v0.1.0` is the distribution at which a stranger, from the command line alone, can do
everything in the list below on a fresh machine, but for what a line says follows the first
release, and nothing in the list requires a window.** Each line is a thing a person can do and
test. A line that follows the first release keeps the name and the meaning given here and is
not required for the tag. The exact flags and output of a verb are the CLI's own reference,
generated from the binary and held to this list by a check; this RFC fixes the nouns, the verbs
and what each does, and a verb's name or meaning changes only by editing this RFC.

### One command tree, two ways in

The CLI and the REPL are one parser. `kotton <verb>` runs one verb; `kotton` alone enters the
REPL, which unlocks once, holds a current drive and directory, adds `cd`, `pwd`, `clear`,
`help` and `exit` (and `quit`, its alias) and a prompt that shows who and where, and otherwise
runs the same verbs without the prefix. Nothing else exists in one that does not exist in the
other, and a check holds the two trees equal. The REPL's syntax, not verbs, is a pipe `|`
between verbs, `;`, `&&` and `||` between commands, and `>` and `>>` into a drive path and
never into `host:`, because inside the REPL there is no shell to supply them; from a terminal
the shell's own do the same over `kotton cat` and `kotton cp -`.

### Conventions

- Files use the words a person already knows, at the top level: `ls`, `cat`, `cp`, `mv`, `rm`,
  `mkdir`, `rmdir`, `touch`, `find`, `grep`, `head`, `tail`, `wc`, `sort`, and beside them
  `edit`, `open`, `trash`, `restore`, and after the first release `diff`, `search`, `versions`.
  A path on a drive is `<drive>:/<path>`. From the shell, a path with no drive is the machine's
  own filesystem, as it is for `scp`; in the REPL, where there is a current drive, a bare path
  is that drive's and `host:` names the machine. `cp` copies between the machine and a drive in
  either direction, copies only what is new or changed when run again, and is the one way a
  folder is put on a drive: `cp -r ~/Photos photos:/`.
- Everything else is `kotton <noun> <verb>`, singular nouns, with the same verbs wherever they
  apply: `create`, `list`, `show`, `rm`, and the noun's own. `list` is for nouns; `ls` is for
  files.
- Output is for a person by default and `--json` on every `list` and `show`; exit codes mean
  something (0 done, 1 refused with the reason on stderr, 2 usage); `cat` and `cp` read and
  write standard input and output, and `grep`, `head`, `tail`, `wc` and `sort` read it, so the
  shell and the REPL's pipe do the rest.
- Text is detected by content, as git detects it, never by extension. How a difference is shown
  follows the first release, with `diff`, and with it how a conflict's two versions are shown
  against each other: text gets line diffs and conflicts a person can read side by side;
  everything else is bytes and gets two named versions to choose between. That a file has a
  conflict, and the way to its other version, do not wait for that: they are `status` and
  `restore <path>`, under Files. The CLI renders no document and ships one editor, a plain text
  editor in the terminal that runs in the process and holds the file in memory, so that the
  ordinary edit puts nothing in the clear on any disk; the machine's own editor is reached by
  `open`.

### What a person can do

**Identity.** `identity create <name>` (a passphrase, once) · `list` · `use` · `show` · `lock`
· `unlock` · `set-passphrase` · `recovery` (add a recovery phrase and print it with the vault
line; a phrase is stored nowhere and is never shown again; an earlier phrase stays able to
recover until a rotation retires it) · `rotate` (new keys under the same identity, from a
phrase, with the contest window the identity page describes; the other phrases are retired by
it, but for one committed before the phrase it is from, which still stands against it in its
holder's hands, RFC 0005) · `recover` (take an identity back on a new machine) · `session
list|end` (the sessions the daemon holds for this identity, and ending one; follows the first
release) · `rm` · `whoami`. `id` is the short form.

**Devices.** `device add` (pair another device by the six digits) · `join <name> <code>` (the
other side) · `list` (with last seen and the drives each keeps whole) · `show` · `rename` · `rm
<device>` (revoke it: refused at its next connection to every drive and device that holds the
revocation, which each learns from the next device that holds it to reach it; if it may be in
someone else's hands the person is told to `identity rotate`, which retires the keys it held
under the identity page's rule) · `bind` (bind this identity to this device's hardware) ·
`receive off|10m|always` (follows the first release, with `send`: whether this device has a
listener and an announcement on the network it is on, so another of the person's devices there
can find it and `send` to it; off until told, since a relay introduces the person's devices
otherwise; `always` is said back at every `status`, and is refused where the platform has no
announcement) · `reset` (start this device fresh).

**Drives.** `drive create <name> [--on <vault>]` · `list` (where each lives, its size, which
devices pin it whole) · `show` · `move <drive> --to <vault>` (under the same key; every device
follows) · `export <drive> <path>` (the whole drive, sealed, to a disk) · `enrol <line>`
(accept a host's vault as a place for drives; it asks nothing else).

**What this device holds.** `pin default whole|on-demand` (this device's default for every
drive it has not been told about: the first device starts at whole, a device paired later at on
demand) · `pin <drive>[:/<folder>] whole|on-demand|default` (the nearest setting wins, folder
over drive over the device's default; `default` follows the one above) · `pin list` (what this
device keeps, its default and its exceptions, and the notice when no device pins a drive
whole). Every device sees every drive; the device that creates a drive pins it whole until told
otherwise, whatever its default; a full device short of room says so and names the setting.
`pin <drive> whole` is "collect all my things". `sync` syncs this device's drives, now. What a
device keeps is its own; a drive carries which devices keep it whole, as RFC 0002 says.

**Files.** `ls` · `cat` · `cp` · `mv` · `rm` · `mkdir` · `rmdir` · `touch` on drive paths,
pinned or not · `find [path] [-name <glob>]` (names under a path) · `grep <pattern> [path...]`
(the content of the paths named, or of what is piped in; text only) · `head` · `tail` · `wc` ·
`sort` · `edit <path>` (the editor in the process; the file is read from the sealed tree into
memory and written back sealed, and Kotton writes nothing to a disk in between; what the machine
pages out of memory is the machine's, as for every process, and the buffer can be locked in
memory as key material already is) · `open <path>` · `restore <path>` (write a conflict's other
version out beside the file, as a conflict copy) · `diff <path>` (follows the first release,
with `versions` and `restore <path> --version <n>`: text, between versions or conflict copies) ·
`search <query>` (follows the first release: names and text, on this device, over what it pins
and the index it fetches, which is a sealed object in the drive like any other) · `versions
<path>` · `restore <path> --version <n>`, both of which follow the first release · `trash
list|restore|empty`. Versions live in the drive and every device sees them, under the store's
one switch per file. A deleted thing stays in the drive's trash for thirty days, then one of the
person's own devices asks the drive to purge it. That is the request from the person's machine
the store requires: `rm` is the person asking for erasure in thirty days, `trash empty` is
asking for it now, and nothing but those two asks. Thirty days is the earliest, since the purge
waits for one of the person's devices to be awake, and compaction waits for it.

A conflict is two of a person's devices each writing one file before it had the other's write.
The drive keeps both versions, whatever the file's switch: one is the file, the same one on
every device, and the other is kept with it in the drive, through every sync and every
compaction. Neither is dropped until the person chooses. `status` names each file that has a
version kept this way, on each device of the person's once it has synced. `restore <path>`
writes the kept version out beside the file as a conflict copy, a file of its own under a name
it says, and erases nothing. From then the two are two files, and which stays is the person's to
say with the file verbs: the one not wanted goes by `rm`, through the trash, as any file does.
Where one device removed the file and another wrote it, the written version is never the one
dropped: it is the file, or it is the kept version and `restore <path>` writes it at the path.
How the two are shown against each other follows the first release, with `diff`.


**Sharing.** `send <path> <device>` (follows the first release: hand a file or folder to one of
my own devices directly, on one network, no relay; the other device's person is asked and says
where in their drives it lands; nothing afterwards) · `share create <path> <contact>` (a sealed
copy that becomes theirs, delivered like a message and landing under "shared with you" in their
tree, as the conversations page has it; the original is never re-keyed, a folder is a copy of
each file in it, and my landlord learns nothing of who holds one) · `share list` (what I have
shared and what was shared with me). A copy cannot be taken back, so there is no `share
revoke`; a folder two people hold as one is a grant the protocol has not shaped, and is not
here.

**Contacts and messages.** `contact pair` · `join <code>` · `list` (every card this person
holds, a removed contact's shown as ended) · `show` · `rm <contact>` (remove them: as a contact,
this person's devices take nothing more from them and send them nothing more, and nothing this
person holds is erased) · `erase <contact>` (erase what this person's devices and drives hold
from or about a contact already removed; refused for one who has not been) · `message send
<contact> <text>` · `message inbox` · `message attach <contact> <path>` (the same sealed copy a
share is, carried in the conversation with that contact) · `message channel
new|send|read|admit|remove|leave` · `message channel attach <channel> <path>` (the same sealed
copy, to each member of the channel).

Removing a contact and erasing what is held from one are two acts. `contact rm` ends a contact
from this person's side and asks for no erasure. The card stays, marked ended: the contact's
identifier, their key, their key log and the name this person gave them. The reach note they
signed is dropped from it: where they stand, their devices' keys, and the invited key to a relay
they operate. The conversation, the shared copies and the entries of `share list` stay as they
were. The ended card is what a device recognises a removed contact by; it is kept until `contact
erase` and is erased by nothing else. From then a removed contact is, for what they may say to
this person, a person never paired with. While the two share no channel, no device of this
person's derives the pair's door or answers at it, and the door is absent from the window a
device next hands its vault. A device hands its vault its doors for the hours ahead whole, and
the vault keeps what it was last handed; so no other request is made for a removal, to a vault
or to a relay, and none names a contact. A window is handed in an order that carries nothing: by
hour and, within an hour, by the door's own label. While the two share a channel, the pair's
door stands, at this person's devices and at their vault, for that channel's copies and for
nothing else: those arrive and are signed for as any member's are, and this person's copies in
that channel go to them as to any member. The door is absent from the next window once the last
channel the two share is left. What a removed contact sends as a contact (a direct message, a
direct share, a reach note) is neither filed nor shown, by `status` or by `message inbox`, and
is not signed for, whenever it was sent, by any device that has followed the removal. Whatever
of it waits at a vault, from before the removal or since, is taken at the next pickup,
recognised by the ended card, and taken off the vault, whether or not the two share a channel.
Where a shared channel keeps the door, an awake device gives it the refusal a member never
paired with is given. The acknowledgement that takes a delivery off the vault is the pickup's to
the vault: it names the delivery by the key made for that one delivery, names nobody, and is not
a receipt, which is a signature under a device key the recipient's reach note lists, returned to
the sender. A delivery a device cannot hold to the card's key, or to a key the card's log
proves, is not acknowledged, and lapses at the vault's own bound. A receipt from them for a
direct message or a direct share this person sent before the removal is among what they send as
a contact. It would be held to the reach note on their card, which went with the removal, and a
shared channel answers for that channel's copies and for nothing else; so it is not believed and
not kept, and this person's copy reads as never signed for, which says what this person's
devices can prove and not what the contact holds. What this person sent the removed contact
directly and no device of theirs signed for stops being sent: it leaves what each device keeps
to send again as that device follows the removal, and a vault carrying it is told by the
ordinary pass that tells it what to carry, which names each message by its own key and names
nobody. This person's own copy stays in the conversation, as never signed for. `message send`,
`message attach` and `share create` to a removed contact, and `message channel admit` naming
one, are refused until the two pair again. As a contact the removed contact is sent nothing
more: no notice of the removal, no further reach note, and no receipt for what they send as a
contact. They keep what they were sent, this person's key and the last reach note this person
signed them. Where no channel is shared, past its expiry they no longer believe the note and
still know what it said. Where one is shared they are a member of it, and a member is told, when
a channel's members change, where each other member stands and their devices' keys, so that the
channel's copies reach them; so for as long as the two share a channel the removed contact goes
on learning that of this person. Where no channel is shared they can find that the door no
longer stands, and about when: it is what they would find if this person had left that relay,
and it is not what a person whose devices sleep looks like, since a vault keeps that person's
doors. No host is told of a removal. The relay this person's devices and vault reserve at sees
one reservation fewer, among labels that name nobody and last an hour. The removed contact's
vault and relay see what they see of any message that is not delivered, and their vault goes on
trying a door that does not stand. What this person's vault learns is on the landlord's row of
the threat model. `contact rm` says which channels the two still share, which `message channel
remove` and `message channel leave` end and removing does not; and, where this person operates a
relay, that the removed contact still holds its invited key until the relay's key is changed,
which removing does not do. The removal is a change to the card, followed by this person's other
devices as any change in their own store is, and a device that has not yet followed it acts on
the card as it was. Pairing with a removed contact again is the ceremony again: it makes the
ended card a contact's card once more, under the same identifier, with a fresh reach note and
everything held before.

`contact erase` is the person's own erasure of what they hold, and ends nothing that `contact
rm` did not. It is refused for a contact who has not been removed, on any device that has not
followed the removal, with the reason, because the card and key are among what it erases, and a
device that had erased the key of a contact it still accepts would have nothing to hold their
next message to. `contact erase` first says what it is about to erase and what it cannot reach,
and erases nothing until the person has said yes, since part of what it erases has no trash. It
erases the direct conversation with that contact, in both directions; the folder of copies from
them under "shared with you"; the entries of `share list` that name them; the ended card, with
their key, their key log and the name; and the lines of `log` that name the contact, the line
`log` keeps of the erasure naming nobody. It reaches every device of this person's, each as it
follows, and the sealed copies a vault keeps of the files. The files go as every deleted file
goes: to the drive's trash for thirty days, under the folder's name and restorable until then,
erased at once by `trash empty`, and purged by the ask the Files line names. So for a file
`contact erase` is `rm`, and the asks for erasure stay two. The conversation, the card and the
lines of `log` are not files on a drive path: they are removed as any record or object of the
person's store is, and have no trash. It does not reach what the contact holds, which is theirs:
every message sent them, every copy shared with them, this person's key and the last reach note;
`contact erase` says so. It does not reach a channel's history, which is the channel's and each
member's, nor the originals of what this person shared, nor what a host has already observed. A
landlord sees of it what it sees of any deletion in a drive it keeps, and is handed no name; a
relay, the contact and the contact's hosts are sent nothing. A deletion that other devices
follow is a marker for as long as the following needs it; the card's marker carries the
contact's identifier, is kept no longer than the store keeps any deletion for other devices to
follow, and is erased with it. That time is finite, and the first release states it. After that
no device of this person's holds the contact's identifier or key outside a channel the two were
in. What still arrives from them cannot be held to a key, is neither filed nor acknowledged, and
lapses at the vault's bound; where a channel is still shared they are a member never paired
with, known by the roster's key; and pairing again makes a new card with no history. Erasing a
conversation with someone who is still a contact is a different act and is not offered. Both
verbs are requests at the person's own daemon and at no other door.

**Records and schemas.** `record list|get|put|rm --schema <name>` · `record import|export
--schema <name> <path>` (an `.ics` calendar comes in this way, after the first release) ·
`schema list|show|register`. Records are typed, sealed, in a drive or, as the log is until
pinned, on the device that made them, and read from every device that holds them: the layer
under contacts and every future app, and what the keychain, the whole-keeper claims and a
pinned `log` are made of.
A schema is declared by whoever ships it and named by the key that publishes it (RFC 0004); none
is Kotton's by right, and one nobody at Kotton has seen works like any other. A grant prompt
shows a schema by its name under the name the person gave its publisher's key, and never names
the publisher by a string the publisher chose. A calendar is an app on records, not a primitive;
Kotton may ship one as one app among any.

**The keychain.** `secret set|get|list|rm|generate` (passwords and tokens as sealed records
with a verb a person reaches for; passkeys wait for a browser or an OS to present them to).

**Host.** `host mint` (the lines that stand up a relay and a vault) · `host enrol <line>` (the
operator completes the key `mint` made, from a relay's line, a vault's, or a host's one line
for both; no digits, since it is their own) · `host list` · `host health` · `host relay
set|clear|status` (which relay this device stands at) · `host relay rule|invite|forget` and
`host vault people|invite|digits|space|drop|stand|move|forget` (operating a relay or a vault of
your own, for the people you admit).

**Seeing it.** `status` (daemon, drives, devices, host, what is syncing, each file that has a
conflict's other version kept, and what arrived: a message, a share and, after the first
release, a transfer offer, delivered to the daemon without polling and shown here and by
`message inbox`; while every device of the person's is asleep a message or a share waits sealed
at their vault until a device of theirs takes it) · `log` (what this device's daemon was asked:
which device or app read or wrote what, when, from where; kept as sealed records on this device,
shown per device, and replicated to a drive only if the person pins it there, so no host learns
a person's rhythm from it; it does not see reads of ciphertext at a host nor what happens to a
copy handed out) · `doctor` (follows the first release: the machine's fitness, and `doctor logs`
for the daemon's own event log, which is about the daemon and not about the person) · `repl`.

### The test of the release

Two people, each at a machine of their own and each with a host of their own, a relay and a
vault they stood up from the lines `host mint` prints. They make first contact at one relay:
one person invites the other to the relay they operate (`host relay invite`), and the other
stands a device at it (`host relay set`). They pair as contacts on a code one reads aloud to
the other. One sends the other a message while every device of the other's is off; it waits
sealed at the recipient's vault and is read when a device of theirs wakes. One shares a file,
and the other opens it from their own tree. The test needs no machine and no party beyond the
two people's own devices and hosts. First contact between two people who stand at different
relays is not settled by this text. The run is made from the release's own archives on fresh
machines, and `v0.1.0` is not tagged before it passes.

### Where it runs

Devices: Linux, macOS and Windows on x86_64, and Linux and macOS on arm64; Windows on arm64 has
never been built and is not promised here. Each is installed by the nine lines (a PowerShell
installer on Windows, verifying the same signature with the OpenSSH the system carries). Each
of the three operating systems is built and tested in the gate on every pull request, Windows
included, so that what a person runs is what the release was held to. Hosts: a Linux machine
the person controls, at home or rented from any provider that sells one (a DigitalOcean droplet
is this, and its S3-shaped object store is a shelf the vault already speaks), stood up by the
lines `host mint` prints. One managed host that sells no machine, Cloudflare, follows the first
release, with the RFC that RFC 0001 reserves for what a vault needs to know: it is the first
adapter under the rule that a vault is its verbs over a transport and a store, with no provider
named outside the adapter. The portability claim is tested between those two, and that test
follows the first release with the adapter: a drive moved from a vault on a rented machine to
the adapter and back, under the same key, every device following. No managed host, no hosted
offering run by Kotton LLC, and no phone, at `v0.1.0`; and no managed host but that one is
named here.

### What leaves the sealed tree, and when

Nothing of a person's is on a disk in the clear unless they said so, once, for that thing. The
ways out, each explicit: `cat` writes to the terminal and never to disk; `cp` from a drive to a
machine path writes plaintext there because the person named the path; `open` hands a plaintext
copy to the machine's own program and says where it put it, and that copy is the person's to
remove; `edit` puts nothing anywhere, because the editor is in the process. What a program
outside Kotton does with a copy is governed by the machine: Kotton's guarantee covers what
Kotton confines. `send`, which follows the first release, lands in the receiving device's
sealed tree and puts nothing in the clear. A confined editor, an app in the sandbox that reads
and writes the sealed tree through the app surface, is the desktop's and the SDK's, and
replaces most uses of `open`; it is named here as the way back and not built here. The session
token on disk is a defect the first release fixes.

### Not in `v0.1.0`, on purpose

The window, the compositor, the desktop and apps a person clicks; inference and models;
organisations, their grants, their issuers and project records; the browser portal and the
gateway, and admitting a phone's browser; a drive mounted as a filesystem so on-demand files
open with anything (the desktop's turn, per operating system); a link for someone without
Kotton; live co-editing; passkeys. Each has its repository or its own RFC, and none changes
what the platform is from the command line.

What follows the first release is decided here and is not required for its tag: `identity
session list|end`; `send` and `device receive`; `search`; `diff`, `versions` and `restore <path>
--version <n>`, and with them how a difference is shown, a conflict's two versions against each
other among them; an `.ics` calendar through `record import`; `doctor` with `doctor logs`; and
the Cloudflare adapter, with a drive's move to it and back and the portability test. Each keeps
the name and the meaning this RFC gives it, and each line of the Decision that names one says so
where it stands. That a file has a conflict, and `restore <path>`, are not among them.

## Consequences

- The command tree is rewritten once, to this list. Every verb in the tree today, the CLI's
  (`docs/cli/reference.md`) and the REPL's registry (`client/shell/src/commands`), has a place
  or leaves, as the table says:

| Today | At `v0.1.0` |
|---|---|
| `login`, `id login` | `identity use` (the daemon keeps the session) |
| `id create/list/use/rm/lock/unlock/set-passphrase/recover` | `identity …`, unchanged |
| `id add-recovery` | `identity recovery` |
| `id rotate` | `identity rotate` |
| `id join` | `device join` |
| `id bind-device` | `device bind` |
| `device add/join/list` | unchanged |
| `device name`, `device revoke`, `device reset` | `device rename`, `device rm`, `device reset` |
| `device admit <vault invite>` | `drive enrol`, the tenant accepting a vault by its invite; `--browser` and `--gateway` leave with the portal |
| `device add --blind` | leaves; the tenant's acceptance is `drive enrol`, by invite |
| `keep` | `cp -r <folder> <drive>:/` |
| `enrol <relay line\|vault line\|host line>` | `host enrol <line>`, the operator completing their own key |
| `cp`, `cat`, `whoami`, `repl` | unchanged; `ls`, `mv`, `rm`, `mkdir`, `rmdir`, `touch` join them at the top, with the six below |
| `ted` | `edit`: the editor in the process, kept because it puts nothing in the clear |
| `door status\|closed\|10m\|always` | `device receive off\|10m\|always`, which follows the first release |
| `session list\|end` | `identity session list\|end`, which follows the first release |
| `logs` | `doctor logs`, which follows the first release; `log` is the access log, a different thing under a near name, said here on purpose |
| `secrets` | `secret` |
| `find`, `grep`, `head`, `tail`, `wc`, `sort` | unchanged, at the top; `search` is the index and a different thing, and follows the first release |
| `date`, `echo`, `env`, `export`, `unset` | leave; they are the shell's, not a file's |
| `daemon` | `status`; and `doctor`, which follows the first release |
| `cd`, `pwd`, `clear`, `help`, `exit`, `quit`; `\|`, `;`, `&&`, `\|\|` | the REPL's own, unchanged; `>` and `>>` join them |
| `sync add/list/remove` | `pin` and `drive`: targets by location are drives, and a device is not one (RFC 0002) |
| `sync` (all paired devices) | `sync`, this device's drives: a device is not a target (RFC 0002) |
| `sync now [--node <id>] [--no-wait]` | `sync`; `--node` leaves with the device as a target |
| `sync [--push\|--pull] <remote>` | `sync`, this device's drives |
| `host:` | kept in the REPL only; from the shell a bare path is the machine's |
| `mint` | `host mint` |
| `relay set/clear/status` | `host relay set/clear/status` |
| `relay rule/invite/forget`, `vault …` | `host relay …`, `host vault …` |
| `peer` | replaced: devices are `device`, nodes are `host`, as RFC 0002 has it |
| `config status`, `net` | `status` |
| `config paths` | `doctor`, which follows the first release |
| `config relay` | `host relay` |
| `grant revoke` (an app's capability envelope) | leaves with the sandbox and the desktop, beside `app` |
| `grant import`, `issuer`, `records init/invite`, `org` | leave with organisations |
| `records import/export`, `import` (`.ics`), `record open` | `record import/export`, `record get`; an `.ics` calendar comes in through `record import` after the first release |
| `contact`, `message`, `schema` | unchanged |
| `app`, `extension`, `handler` | leave with the sandbox and the desktop |
| `model`, `config inference` | leave with inference |

- A check holds both trees to this list: every clap subcommand and every verb the REPL's
  registry registers is either in the Decision or in a "Today" cell of the table, and nothing
  in the Decision is absent from the binary, but for a verb the Decision says follows the first
  release, which the first release need not carry; the REPL's tree is held equal to the CLI's.
- `v0.1.0` is tagged when three things are true, and not before. A stranger can test what the
  first release holds: identity and recovery; devices; drives, `pin` and `sync`; the file verbs,
  `restore <path>` among them; `share`; contacts, with `contact rm` and `contact erase`;
  messages, channels and both `attach` verbs; records and schemas; the keychain; `log`, `host`
  and `status`; each on the three operating systems the gate builds and tests; and with them
  every other line above that does not say it follows the first release. The test of the release
  has passed from the release's own archives. And RFC 0005's handshake is the way in at every
  connection to a drive, between a person's devices and at a contact's door. When the work lands
  is the distribution's tracking issue's to say, not this RFC's.
- Contacts, messages, channels and sharing are the user daemon's. They seal and sign with the
  identity's key, which no other process holds, so they have no repository of their own, and
  they ship in the system archive and in no other: a vault and a relay link none of them.
- Removing a contact sends no host anything that names them, and erases nothing; erasing what is
  held from a contact is the person's own erasure of what they hold. A person who has ended a
  contact holds a record that they once paired with them. It is sealed in their own store, a
  host sees none of it, and erasing is the act that ends it; no one else holds it to be asked
  for.
- A vault that keeps doors for its tenant keeps them in a number that follows the tenant's
  contacts, so the number moves when a contact is made or removed, at the window that carries
  the change. A vault that keeps doors and carries messages can also tell which of the doors,
  deliveries and carried messages it holds are one pair's, within a week and from one week to
  the next, and never whose. Both are conceded for the first release, which does not pad the
  number and states no rule against the second, and both are on the landlord's row of the threat
  model. The ways back are doors held in a number that does not follow the contacts, and doors
  and carried messages handed so that a vault cannot tie one hour's to another's or a message to
  a door; they are named here and not built here.
- Two devices writing one file lose the person nothing at the first release: both versions are
  kept, `status` says so, and `restore <path>` takes the other out as a file. Reading the two
  against each other waits for `diff`.
- "Enablement, never grievance" has a verb: `log` shows a person what their own daemon was
  asked, which is the thesis made visible without making the asking visible to anyone else.

## Reaches

- `kotton`: the daemon's side of every verb above, and a gate that builds and tests Windows.
- `kotton-cli`: the command tree, the REPL, the generated reference and the check that holds it
  to this list.
- `kotton-protocol`: the request types the new verbs cross with (the trash, the access log's
  query, the shared copy and the attachment, the notification, a conflict's kept version,
  removing a contact and erasing what is held from one), in `v0.1.0` or a patch after it; the
  keychain and the access log as record schemas.
- `kotton-dist`: the nine lines and the guide, rewritten to the tree; the PowerShell installer;
  `v0.1.0` itself.
- `kotton-vault` and `kotton-relay` when cut, and the Cloudflare adapter when written; the
  adapter follows the first release and waits on the RFC that RFC 0001 reserves for what a
  vault needs to know, since "a vault is its verbs over a transport and a store" is that
  RFC's to settle, and stated here as the rule the adapter is held to.
- RFC 0002, edited in place (no tag exists): the per-device choice is set with `pin`.
