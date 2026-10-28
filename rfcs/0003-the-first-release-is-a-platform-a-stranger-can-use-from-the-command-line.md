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
everything in the list below on a fresh machine, and nothing in the list requires a window.**
Each line is a thing a person can do and test. The exact flags and output of a verb are the
CLI's own reference, generated from the binary and held to this list by a check; this RFC fixes
the nouns, the verbs and what each does, and a verb's name or meaning changes only by editing
this RFC.

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
  `edit`, `open`, `diff`, `search`, `versions`, `restore`, `trash`. A
  path on a drive is `<drive>:/<path>`. From the shell, a path with no drive is the machine's
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
- Text is detected by content, as git detects it, never by extension: text gets line diffs and
  conflicts a person can read side by side; everything else is bytes and gets two named
  versions to choose between. The CLI renders no document and ships one editor, a plain text
  editor in the terminal that runs in the process and holds the file in memory, so that the
  ordinary edit puts nothing in the clear on any disk; the machine's own editor is reached by
  `open`.

### What a person can do

**Identity.** `identity create <name>` (a passphrase, once) · `list` · `use` · `show` · `lock`
· `unlock` · `set-passphrase` · `recovery` (add a recovery phrase and print it with the vault
line; a phrase is stored nowhere and is never shown again; an earlier phrase stays able to
recover until a rotation retires it) · `rotate` (new keys under the same identity, from a
phrase, with the contest window the identity page describes; earlier phrases are retired by it)
· `recover` (take an identity back on a new machine) · `session list|end` (the sessions the
daemon holds for this identity, and ending one) · `rm` · `whoami`. `id` is the short form.

**Devices.** `device add` (pair another device by the six digits) · `join <name> <code>` (the
other side) · `list` (with last seen and what each pins) · `show` · `rename` · `rm <device>`
(revoke it: refused at its next connection to every drive and device; if it may be in someone
else's hands the person is told to `identity rotate`, which retires the keys it held under the
identity page's rule) · `bind` (bind this identity to this device's hardware) · `receive
off|10m|always` (whether this device has a listener and an announcement on the network it is
on, so another of the person's devices there can find it and `send` to it; off until told,
since a relay introduces the person's devices otherwise; `always` is said back at every
`status`, and is refused where the platform has no announcement) · `reset` (start this device
fresh).

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
device pins is sealed records, as RFC 0002 says: its default among the person's identity
records, and each exception in the drive it is about.

**Files.** `ls` · `cat` · `cp` · `mv` · `rm` · `mkdir` · `rmdir` · `touch` on drive paths,
pinned or not · `find [path] [-name <glob>]` (names under a path) · `grep <pattern> [path...]`
(the content of the paths named, or of what is piped in; text only) · `head` · `tail` · `wc` ·
`sort` · `edit <path>` (the editor in the process; the file is read from the sealed tree into
memory and written back sealed, and Kotton writes nothing to a disk in between; what the
machine pages out of memory is the machine's, as for every process, and the buffer can be
locked in memory as key material already is) · `open <path>` · `diff <path>` (text: between
versions or conflict copies) · `search <query>` (names and text, on this device, over what it
pins and the index it fetches, which is a sealed object in the drive like any other) ·
`versions <path>` · `restore <path> --version <n>` · `trash list|restore|empty`. Versions live
in the drive and every device sees them, under the store's one switch per file. A deleted thing
stays in the drive's trash for thirty days, then one of the person's own devices asks the drive
to purge it. That is the request from the person's machine the store requires: `rm` is the
person asking for erasure in thirty days, `trash empty` is asking for it now, and nothing but
those two asks. Thirty days is the earliest, since the purge waits for one of the person's
devices to be awake, and compaction waits for it.


**Sharing.** `send <path> <device>` (hand a file or folder to one of my own devices directly,
on one network, no relay; the other device's person is asked and says where in their drives it
lands; nothing afterwards) · `share create <path> <contact>` (a sealed copy that becomes
theirs, delivered like a message and landing under "shared with you" in their tree, as the
conversations page has it; the original is never re-keyed, a folder is a copy of each file in
it, and my landlord learns nothing of who holds one) · `share list` (what I have shared and
what was shared with me). A copy cannot be taken back, so there is no `share revoke`; a folder
two people hold as one is a grant the protocol has not shaped, and is not here.

**Contacts and messages.** `contact pair` · `join <code>` · `list` · `show` · `rm` · `message
send <contact> <text>` · `inbox` · `channel new|send|read|admit|remove|leave` · `message attach
<path>` (the same sealed copy a share is, carried in a message).

**Records and schemas.** `record list|get|put|rm --schema <name>` · `record import|export
--schema <name> <path>` (an `.ics` calendar comes in this way) · `schema list|show|register`.
Records are typed, sealed, in a drive, read from every device: the layer under contacts and
every future app, and what the keychain, the pins and a pinned `log` are made of. A schema is
declared by whoever ships it; none is Kotton's by right, and one nobody at
Kotton has seen works like any other. A calendar is an app on records, not a primitive; Kotton
may ship one as one app among any.

**The keychain.** `secret set|get|list|rm|generate` (passwords and tokens as sealed records
with a verb a person reaches for; passkeys wait for a browser or an OS to present them to).

**Host.** `host mint` (the lines that stand up a relay and a vault) · `host enrol <line>` (the
operator completes the key `mint` made, from a relay's line, a vault's, or a host's one line
for both; no digits, since it is their own) · `host list` · `host health` · `host relay
set|clear|status` (which relay this device stands at) · `host relay rule|invite|forget` and
`host vault people|invite|digits|space|drop|stand|move|forget` (operating a relay or a vault of
your own, for the people you admit).

**Seeing it.** `status` (daemon, drives, devices, host, what is syncing, and what arrived: a
message, a share, a transfer offer, delivered to the daemon without polling, held sealed by the
drive while a device is asleep as messages are, and shown here and by `inbox`) · `log` (what
this device's daemon was asked: which device or app read or wrote what, when, from where; kept
as sealed records on this device, shown per device, and replicated to a drive only if the
person pins it there, so no host learns a person's rhythm from it; it does not see reads of
ciphertext at a host nor what happens to a copy handed out) · `doctor` (the machine's fitness,
and `doctor logs` for the daemon's own event log, which is about the daemon and not about the
person) · `repl`.

### Where it runs

Devices: Linux, macOS and Windows on x86_64, and Linux and macOS on arm64; Windows on arm64 has
never been built and is not promised here. Each is installed by the nine lines (a PowerShell
installer on Windows, verifying the same signature with the OpenSSH the system carries). Each
of the three operating systems is built and tested in the gate on every pull request, Windows
included, so that what a person runs is what the release was held to. Hosts: a Linux machine
the person controls, at home or rented from any provider that sells one (a DigitalOcean droplet
is this, and its S3-shaped object store is a shelf the vault already speaks), stood up by the
lines `host mint` prints; and one managed host that sells no machine, Cloudflare, as the first
adapter under the rule that a vault is its verbs over a transport and a store, with no provider
named outside the adapter. The portability claim is tested between those two: a drive moved
from a vault on a rented machine to the adapter and back, under the same key, every device
following. No second managed host, no hosted offering run by Kotton LLC, and no phone, at
`v0.1.0`.

### What leaves the sealed tree, and when

Nothing of a person's is on a disk in the clear unless they said so, once, for that thing. The
ways out, each explicit: `cat` writes to the terminal and never to disk; `cp` from a drive to a
machine path writes plaintext there because the person named the path; `open` hands a plaintext
copy to the machine's own program and says where it put it, and that copy is the person's to
remove; `edit` puts nothing anywhere, because the editor is in the process. What a program
outside Kotton does with a copy is governed by the machine: Kotton's guarantee covers what
Kotton confines. `send` lands in the receiving device's sealed tree and puts nothing in the
clear. A confined editor, an app in the sandbox that reads and writes the sealed tree through
the app surface, is the desktop's and the SDK's, and replaces most uses of `open`; it is named
here as the way back and not built here. The session token on disk is a defect the first
release fixes.

### Not in `v0.1.0`, on purpose

The window, the compositor, the desktop and apps a person clicks; inference and models;
organisations, their grants, their issuers and project records; the browser portal and the
gateway, and admitting a phone's browser; a drive mounted as a filesystem so on-demand files
open with anything (the desktop's turn, per operating system); a link for someone without
Kotton; live co-editing; passkeys. Each has its repository or its own RFC, and none changes
what the platform is from the command line.

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
| `door status\|closed\|10m\|always` | `device receive off\|10m\|always` |
| `session list\|end` | `identity session list\|end` |
| `logs` | `doctor logs`; `log` is the access log, a different thing under a near name, said here on purpose |
| `secrets` | `secret` |
| `find`, `grep`, `head`, `tail`, `wc`, `sort` | unchanged, at the top; `search` is the index and a different thing |
| `date`, `echo`, `env`, `export`, `unset` | leave; they are the shell's, not a file's |
| `daemon` | `status` and `doctor` |
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
| `config paths` | `doctor` |
| `config relay` | `host relay` |
| `grant revoke` (an app's capability envelope) | leaves with the sandbox and the desktop, beside `app` |
| `grant import`, `issuer`, `records init/invite`, `org` | leave with organisations |
| `records import/export`, `import` (`.ics`), `record open` | `record import/export`, `record get` |
| `contact`, `message`, `schema` | unchanged |
| `app`, `extension`, `handler` | leave with the sandbox and the desktop |
| `model`, `config inference` | leave with inference |

- A check holds both trees to this list: every clap subcommand and every verb the REPL's
  registry registers is either in the Decision or in a "Today" cell of the table, and nothing
  in the Decision is absent from the binary; the REPL's tree is held equal to the CLI's.
- `v0.1.0` is tagged when every line above is testable by a stranger, and not before. When
  the work lands is the distribution's tracking issue's to say, not this RFC's.
- "Enablement, never grievance" has a verb: `log` shows a person what their own daemon was
  asked, which is the thesis made visible without making the asking visible to anyone else.

## Reaches

- `kotton`: the daemon's side of every verb above, and a gate that builds and tests Windows.
- `kotton-cli`: the command tree, the REPL, the generated reference and the check that holds it
  to this list.
- `kotton-protocol`: the request types the new verbs cross with (the trash, the access log's
  query, the shared copy and the attachment, the notification), in `v0.1.0` or a patch after
  it; the keychain and the access log as record schemas.
- `kotton-dist`: the nine lines and the guide, rewritten to the tree; the PowerShell installer;
  `v0.1.0` itself.
- `kotton-vault` and `kotton-relay` when cut, and the Cloudflare adapter when written; the
  adapter waits on the RFC that RFC 0001 reserves for what a vault needs to know, since "a
  vault is its verbs over a transport and a store" is that RFC's to settle, and stated here
  as the rule the adapter is held to.
- RFC 0002, edited in place (no tag exists): the per-device choice is set with `pin`.
