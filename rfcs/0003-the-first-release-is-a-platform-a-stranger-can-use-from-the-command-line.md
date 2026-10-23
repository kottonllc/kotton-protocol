# 0003: The first release is a platform a stranger can use from the command line

## Context

RFC 0001 says what the product is made of and RFC 0002 says where a person's things live.
Neither says what `v0.1.0` is: which things a person can do with it, from what surface, and
what it does not yet do. The versioning rule reserves `v0.1.0` for the distribution a person
installs, after the protocol's golden-encoding check exists. Snapshots (`v0.1.0-dev.N`) prove
the pipeline before that and promise nothing.

The reference holds most of a platform already: identities and devices, a sealed store with
versions, drives on vaults, contacts and messages, typed records under schemas, grants, a host
a person mints and enrols, recovery from words. It holds them under the command names that grew
with them, in a command tree where some verbs live at the top and some only inside the REPL,
and where the store is reached by its own words (`keep`) rather than by the words a person
already knows for a filesystem.

What, then, is `v0.1.0`: what a person can do, how the verbs are named, what leaves the sealed
tree and when, and what is deliberately not in it?

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
  path on a drive is `<drive>:/<path>`; a bare path is the current drive's. `cp` copies between
  the machine's filesystem and a drive in either direction, so putting a folder on a drive is
  `cp -r ~/Photos photos:/` and no other word is taught for it.
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
· `unlock` · `set-passphrase` · `recovery` (show or regenerate the words and the vault line) ·
`recover` (take an identity back on a new machine) · `rm` · `whoami`. `id` is the short form.

**Devices.** `device add` (pair another device by the six digits) · `join <name> <code>` (the
other side) · `list` (with last seen and what each pins) · `show` · `rm` (refused at its next
connection) · `lock <device>` (lock a lost device's keys from here).

**Drives.** `drive create <name> [--on <vault>]` · `list` (where each lives, its size, which
devices pin it whole) · `show` · `move <drive> --to <vault>` (under the same key; every device
follows) · `export <drive> <path>` (the whole drive, sealed, to a disk) · `enrol <line>`
(accept a host's vault as a place for drives; it asks nothing else).

**What this device holds.** `pin <drive>[:/<folder>] whole|on-demand` · `pin list` (and the
notice when no device pins a drive whole). The device that creates a drive pins it whole until
told otherwise; a device paired later pins nothing until told; a full device short of room says
so and names the setting. `pin <drive> whole` is "collect all my things". `sync` syncs this
device's drives, now.

**Files.** `ls` · `cat` · `cp` · `mv` · `rm` · `mkdir` on drive paths, pinned or not · `edit
<path>` (decrypt to a private temporary, the person's `$EDITOR`, re-seal, wipe, and say so) ·
`open <path>` (hand a plaintext copy to the machine, explicitly, and say where) · `diff <path>`
(text: between versions or conflict copies) · `search <query>` (names and text, on this device,
over what it pins and the index it fetches) · `versions <path>` · `restore <path> --version
<n>` · `trash list|restore|empty` (deleted things, restorable for a window).

**Sharing.** `send <path> <device>` (hand a file or folder to one of my own devices directly,
on one network, no relay; the other device's person is asked and says where it lands; nothing
afterwards) · `share create <folder> <contact> [--rw]` (a contact sees it among their drives) ·
`share list` · `share revoke`.

**Contacts and messages.** `contact pair` · `join <code>` · `list` · `show` · `rm` · `message
send <contact> <text>` · `inbox` · `channel new|send|read|admit|remove|leave` · `message attach
<path>` (by reference to a shared folder, never a copy).

**Records and schemas.** `record list|get|put|rm --schema <name>` (typed, sealed records in a
drive, read from every device; the layer under contacts and every future app) · `schema
list|show|register` (a schema is declared by whoever ships it; none is Kotton's by right, and
one nobody at Kotton has seen works like any other).

**The keychain.** `secret set|get|list|rm|generate` (passwords, passkeys, tokens: sealed
records with a verb a person reaches for).

**Host.** `host mint` (the lines that stand up a relay and a vault) · `host list` · `host
health` · `host relay set|clear` (which relay this device stands at).

**Seeing it.** `status` (daemon, drives, devices, host, what is syncing) · `log` (every read
and write of the person's things: which device or app, what, when, from where, held by the
log's own verification) · `doctor` (the machine's fitness) · `repl`.

### What leaves the sealed tree, and when

Nothing of a person's is on a disk in the clear unless they said so, once, for that thing.
`cat` writes to the terminal and never to disk. `edit` opens a private temporary for the length
of one edit and wipes it; the window in which a crash or the machine's own indexer could see it
is stated, not hidden. `open` hands a plaintext copy to the machine's own programs because the
person asked, and says where it put it. What a program outside Kotton does with a copy is
governed by the machine, not by Kotton: Kotton's guarantee covers what Kotton runs. A confined
editor, an app in the sandbox that reads and writes the sealed tree through the app surface
with no plaintext on any disk, replaces `edit` and most uses of `open`; it is the desktop's and
the SDK's, and this RFC names it as the way back without building it. The session token on disk
is a defect the first release fixes.

### Not in `v0.1.0`, on purpose

The window, the compositor, the desktop and apps a person clicks; inference; organisations; the
browser portal and the gateway; a drive mounted as a filesystem so on-demand files open with
anything (the desktop's turn, per operating system); a link for someone without Kotton; live
co-editing. Each has its repository or its own RFC, and none changes what the platform is from
the command line.

## Consequences

- The command tree is rewritten once, to this list, before the first snapshot that a stranger
  is asked to try: `keep` becomes `cp`, `enrol` becomes `drive enrol`, `id join` becomes
  `device join`, `config` and `net` fold into `status` and `host`, `app`, `extension`,
  `handler`, `model`, `org`, `issuer` and `peer` leave the tree for their repositories' turns.
  The nine lines change with it, and the guide that teaches them.
- A check holds the CLI's generated reference to this list: a verb in the binary and not here,
  or here and not in the binary, fails lint; the REPL's tree is held equal to the CLI's.
- Each group above lands in a sprint: identity, devices, drives, pins and files with the drive
  model; sharing's `send` with the direct transfer; `share`, messages, records, the keychain
  and `log` after; `v0.1.0` is tagged when every line is testable by a stranger.
- "Enablement, never grievance" has a verb: `log` shows a person every ask made of their
  things, which is the thesis made visible.

## Reaches

- `kotton`: the daemon's side of every verb above, the trash and search, the access log, the
  remote lock, the session token fix.
- `kotton-cli`: the command tree, the REPL, the generated reference and the check that holds it
  to this list.
- `kotton-protocol`: the request types the new verbs cross with (the remote lock, the trash,
  the access log's query, the share's grant shape), in `v0.1.0` or a patch after it; the
  keychain and the access log as record schemas.
- `kotton-dist`: the nine lines and the guide, rewritten to the tree; `v0.1.0` itself.
- RFC 0002, edited in place (no tag exists): the per-device choice is set with `pin`.

