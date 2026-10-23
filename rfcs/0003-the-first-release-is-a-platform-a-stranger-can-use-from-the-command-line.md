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
REPL, which unlocks once, holds a current drive and directory, adds `cd` and `pwd` and a prompt
that shows who and where, and otherwise runs the same verbs without the prefix. Nothing exists
in one that does not exist in the other, and a check holds the two trees equal.

### Conventions

- Files use the words a person already knows, at the top level: `ls`, `cat`, `cp`, `mv`, `rm`,
  `mkdir`, and beside them `edit`, `open`, `diff`, `search`, `versions`, `restore`, `trash`. A
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
  write standard input and output, so the shell does the rest.
- Text is detected by content, as git detects it, never by extension: text gets line diffs and
  conflicts a person can read side by side; everything else is bytes and gets two named
  versions to choose between. The CLI renders nothing and ships no editor.

### What a person can do

**Identity.** `identity create <name>` (a passphrase, once) · `list` · `use` · `show` · `lock`
· `unlock` · `set-passphrase` · `recovery` (add a recovery phrase and print it with the vault
line; a phrase is stored nowhere and is never shown again; an earlier phrase stays able to
recover until a rotation retires it) · `rotate` (new keys under the same identity, from a
phrase, with the contest window the identity page describes; earlier phrases are retired by it)
· `recover` (take an identity back on a new machine) · `rm` · `whoami`. `id` is the short form.

**Devices.** `device add` (pair another device by the six digits) · `join <name> <code>` (the
other side) · `list` (with last seen and what each pins) · `show` · `rename` · `rm <device>`
(revoke it: refused at its next connection to every drive and device; if it may be in someone
else's hands the person is told to `identity rotate`, which retires the keys it held under the
identity page's rule) · `bind` (bind this identity to this device's hardware) · `reset` (start
this device fresh).

**Drives.** `drive create <name> [--on <vault>]` · `list` (where each lives, its size, which
devices pin it whole) · `show` · `move <drive> --to <vault>` (under the same key; every device
follows) · `export <drive> <path>` (the whole drive, sealed, to a disk) · `enrol <line>`
(accept a host's vault as a place for drives; it asks nothing else).

**What this device holds.** `pin <drive>[:/<folder>] whole|on-demand` · `pin list` (and the
notice when no device pins a drive whole). The device that creates a drive pins it whole until
told otherwise; a device paired later pins nothing until told; a full device short of room says
so and names the setting. `pin <drive> whole` is "collect all my things". `sync` syncs this
device's drives, now. What a device pins is a sealed record in the drive, as RFC 0002 says.

**Files.** `ls` · `cat` · `cp` · `mv` · `rm` · `mkdir` on drive paths, pinned or not · `edit
<path>` · `open <path>` · `diff <path>` (text: between versions or conflict copies) · `search
<query>` (names and text, on this device, over what it pins and the index it fetches) ·
`versions <path>` · `restore <path> --version <n>` · `trash list|restore|empty`. Versions live
in the drive and every device sees them, under the store's one switch per file; today a version
is one device's, so this is store work the drive model asks for. A deleted thing stays in the
drive's trash for thirty days; after that the person's own device asks the drive to purge it,
since only a request from the person's machine purges, and compaction waits for the window.

**Sharing.** `send <path> <device>` (hand a file or folder to one of my own devices directly,
on one network, no relay; the other device's person is asked and says where in their drives it
lands; nothing afterwards) · `share create <folder> <contact> [--rw]` (a contact sees it among
their drives) · `share list` · `share revoke`.

**Contacts and messages.** `contact pair` · `join <code>` · `list` · `show` · `rm` · `message
send <contact> <text>` · `inbox` · `channel new|send|read|admit|remove|leave` · `message attach
<path>` (by reference to a shared folder, never a copy).

**Records and schemas.** `record list|get|put|rm --schema <name>` · `record import|export
--schema <name> <path>` (an `.ics` calendar comes in this way) · `schema list|show|register`.
Records are typed, sealed, in a drive, read from every device: the layer under contacts and
every future app. Today records reach only a person's own devices; their replication to a drive
is store work the drive model asks for, and the keychain, the pins and a pinned `log` rest on
it. A schema is declared by whoever ships it; none is Kotton's by right, and one nobody at
Kotton has seen works like any other. A calendar is an app on records, not a primitive; Kotton
may ship one as one app among any.

**The keychain.** `secret set|get|list|rm|generate` (passwords and tokens as sealed records
with a verb a person reaches for; passkeys wait for a browser or an OS to present them to).

**Host.** `host mint` (the lines that stand up a relay and a vault) · `host list` · `host
health` · `host relay set|clear|status` (which relay this device stands at) · `host relay
rule|invite|forget` and `host vault people|invite|digits|space|drop|stand|move|forget`
(operating a relay or a vault of your own, for the people you admit).

**Seeing it.** `status` (daemon, drives, devices, host, what is syncing, and what arrived: a
message, a share, a transfer offer, delivered to the daemon without polling, held sealed by the
drive while a device is asleep as messages are, and shown here and by `inbox`) · `log` (what
this device's daemon was asked: which device or app read or wrote what, when, from where; kept
as sealed records on this device, shown per device, and replicated to a drive only if the
person pins it there, so no host learns a person's rhythm from it; it does not see reads of
ciphertext at a host nor what happens to a copy handed out) · `doctor` (the machine's fitness)
· `repl`.

### Where it runs

Devices: Linux, macOS and Windows, on x86_64, and arm64 where the release builds it, installed
by the nine lines (a PowerShell installer on Windows, verifying the same signature with the
OpenSSH the system carries). The code is written for all three: the socket, keychain and
sandbox providers each have a Windows form. It is built and run in a gate on Linux; it has been
built and run on Windows by hand and not in any gate, and the check that holds Windows today is
a grep for Unix-only calls, because `ring` needs a Windows toolchain to compile. From the
second sprint a Windows runner builds and tests the tree on every pull request, so that what
the owner uses every day is what the release is held to. Hosts: a Linux machine the person
controls, at home or rented from any provider that sells one (a DigitalOcean droplet is this,
and its S3-shaped object store is a shelf the vault already speaks), stood up by the lines
`host mint` prints; and one managed host that sells no machine, Cloudflare, as the first
adapter under the rule that a vault is its verbs over a transport and a store, with no provider
named outside the adapter. The portability claim is tested between those two: a drive moved
from a vault on a rented machine to the adapter and back, under the same key, every device
following. No second managed host, no hosted offering run by Kotton LLC, and no phone, at
`v0.1.0`.

### What leaves the sealed tree, and when

Nothing of a person's is on a disk in the clear unless they said so, once, for that thing. The
ways out, each explicit: `cat` writes to the terminal and never to disk; `cp` from a drive to a
machine path writes plaintext there because the person named the path; `open` hands a plaintext
copy to the machine's own program and says where it put it; `edit` writes a private temporary
(a directory the daemon makes, readable by the person alone) for the length of one edit, hands
it to the person's `$EDITOR`, re-seals the result and removes the temporary. Removing is not
erasing on a modern filesystem, and an editor's own swap or backup files are the editor's;
`edit` says so once, and a temporary left by a crash is reported by `status` until it is
removed. `send` lands in the receiving device's sealed tree and puts nothing in the clear. What
a program outside Kotton does with a copy is governed by the machine: the person's editor is
the machine's program, launched for them, and Kotton's guarantee covers what Kotton confines. A
confined editor, an app in the sandbox that reads and writes the sealed tree through the app
surface with no plaintext on any disk, replaces `edit` and most uses of `open`; it is the
desktop's and the SDK's, named here as the way back and not built here. The session token on
disk is a defect the first release fixes.

### Not in `v0.1.0`, on purpose

The window, the compositor, the desktop and apps a person clicks; inference and models;
organisations, their grants, their issuers and project records; the browser portal and the
gateway, and admitting a phone's browser; a drive mounted as a filesystem so on-demand files
open with anything (the desktop's turn, per operating system); a link for someone without
Kotton; live co-editing; passkeys. Each has its repository or its own RFC, and none changes
what the platform is from the command line.

## Consequences

- The command tree is rewritten once, to this list, before the first snapshot a stranger is
  asked to use the platform on (the walk, not the install test). Every verb in the tree today
  has a place or leaves, as the table says:

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
| `device admit <vault invite>` | `drive enrol`; `--browser` and `--gateway` leave with the portal |
| `keep` | `cp -r <folder> <drive>:/` |
| `enrol` | `drive enrol` |
| `cp`, `cat`, `whoami`, `repl` | unchanged; `ls`, `mv`, `rm`, `mkdir` join them at the top |
| `sync add/list/remove` | `pin` and `drive`: targets by location are drives, and a device is not one (RFC 0002) |
| `sync [--push\|--pull] <remote>` | `sync`, this device's drives |
| `host:` | kept in the REPL only; from the shell a bare path is the machine's |
| `mint` | `host mint` |
| `relay set/clear/status` | `host relay set/clear/status` |
| `relay rule/invite/forget`, `vault …` | `host relay …`, `host vault …` |
| `peer` | replaced: devices are `device`, nodes are `host`, as RFC 0002 has it |
| `config status`, `net` | `status` |
| `config paths` | `doctor` |
| `config relay` | `host relay` |
| `grant revoke` | `share revoke` |
| `grant import`, `issuer`, `records init/invite`, `org` | leave with organisations |
| `records import/export`, `import` (`.ics`), `record open` | `record import/export`, `record get` |
| `contact`, `message`, `schema` | unchanged |
| `app`, `extension`, `handler` | leave with the sandbox and the desktop |
| `model`, `config inference` | leave with inference |

- A check holds the CLI's generated reference to this list: a verb in the binary and not here,
  or here and not in the binary, fails lint; the REPL's tree is held equal to the CLI's.
- The groups land in sprints, and the sprint plan in the distribution's tracking issue is
  amended to match: the Windows runner in the gate and identity, devices, drives, pins, files
  and `status` with the drive model; `send` with the direct transfer; `share`, messages,
  records, the keychain, `log`, `trash`, `search` and the Cloudflare adapter after; `v0.1.0` is
  tagged when every line is testable by a stranger, which is the walk.
- "Enablement, never grievance" has a verb: `log` shows a person what their own daemon was
  asked, which is the thesis made visible without making the asking visible to anyone else.

## Reaches

- `kotton`: the daemon's side of every verb above; versions that travel with the drive and
  records that replicate to it (store work); the trash and search; the access log;
  notifications delivered to the daemon and held by the drive; the Windows runner in the gate;
  the session token fix.
- `kotton-cli`: the command tree, the REPL, the generated reference and the check that holds it
  to this list.
- `kotton-protocol`: the request types the new verbs cross with (the trash, the access log's
  query, the share's grant shape, the notification), in `v0.1.0` or a patch after it; the
  keychain and the access log as record schemas.
- `kotton-dist`: the nine lines and the guide, rewritten to the tree; the PowerShell installer;
  the sprint plan in its tracking issue; `v0.1.0` itself.
- `kotton-vault` and `kotton-relay` when cut, and the Cloudflare adapter when written.
- RFC 0002, edited in place (no tag exists): the per-device choice is set with `pin`.
