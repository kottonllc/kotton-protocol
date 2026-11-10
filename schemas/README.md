# Schemas

The record schemas of Kotton's core set: the three the first release's decision (RFC 0003)
names, the keychain, the whole-keeper claim RFC 0002 defines, and the access log; and the rest
of the reference's core set, today's definitions republished under the release key so that the
core set is one directory (RFC 0004). Each file here is the authority for its schema, in the
form a schema is registered in: its publisher, the key that publishes it; its name under that
key; its revision; its fields with their types; and a description (RFC 0004). A schema's
identity is `<publisher>/<name>`, and its version is the digest of its definition in canonical
form (RFC 0004); the file holds the definition indented for reading, and carries no digest of
itself. The files under `kotton/` are published under Kotton's release key, the key in the
reference's `infra/release/allowed_signers`. From the first tag on they carry that key's
`signature`; before it they carry none, and are what every page here is, a change the maintainer
signed off. An implementation holds these in its core set, unchanged; the reference holds them
in `kotton_schemas`'s core set. A change to a field is a new revision, and a new digest, of the
same schema.

Every record is sealed under its person's key, in a drive or on a device, and a vault or a
relay reads none of it. No field of `keeps-whole`, `secret` or `access-log` is an identifier
minted for the record, and a record's own id is the implementation's.

## `keeps-whole`

That one device keeps one drive whole, as RFC 0002's types say: `device`, the identifier of the
key the device presents, and `drive`, the drive's id. That the record exists is the claim. It
is written in the drive it names, by that device alone, and deleted when the device stops
keeping the drive whole.

## `secret`

A password or a token in the person's keychain, behind `secret`: `label`, what the person calls
it; `value`, the secret itself; `kind`, `password` or `token`; and `note`, anything else the
person writes beside it, or nothing. No app reaches it: a schema grant on it is refused, and it
is kept out of record search.

## `access-log`

One thing a device's daemon was asked, behind `log`. It is kept on that device, and in a drive
only when the person pins the log there.

- `device`: the device whose daemon was asked, by the identifier of the key it presents, so
  that a log kept in a drive is still shown per device.
- `at`: when, to the second.
- `asker`: who asked: an app by its manifest id, which is the same on every device, one of the
  person's devices by the identifier of the key it presents, or this device itself when the
  person asked at its own terminal.
- `door`: what the request came through: `terminal`, `app` or `device`. It is never a network
  address.
- `act`: `read` or `write`.
- `what`: what was read or written, as the person names it: a drive path (`<drive>:/<path>`) or
  a record (`kotton://records/<publisher>/<name>/<id>`). It is never a digest of the content.
  `<drive>` is the drive's name as the person sees it, not its id; a renamed drive's earlier
  entries keep the old name.

No app reaches it: a schema grant on it is refused, as on `secret`, and it is kept out of
record search, so only the person's own CLI or shell
reads it.

## `calendar`

An event, at revision 2: `title`, what the person calls it; `start_at` and `end_at`, when it
starts and ends; `participants`, a list of who takes part, empty when no one is named; and
`location`, where, or nothing. Revision 2 adds what an event brought in from another calendar
carries, each of them or nothing: `all_day`, whether the event is a whole day, in which case
`end_at` is the day after its last, as an `.ics` writes it; `time_zone`, the zone its times
were written in; `description`; `recurrence`, the rule by which it repeats, and
`recurrence_id`, which occurrence of a series this record moves; and `uid`, the identifier the
event's origin gave it. `uid` is the one field here that joins this record to a copy someone
else holds: it is what makes a second import of one file change nothing, and it is sealed
with the record like every other field. Where a file came from and when it was read are not
kept.

## `channel.key-grant`

One epoch of a channel's symmetric key, given to one member, encrypted to the public key of
that member's DID by X25519 ECDH: `channel_did`, the channel's DID; `to_did`, the member's DID;
`enc_key`, the encrypted key; `nonce` and `ephemeral_pub`, the nonce and the ephemeral public
key it was encrypted with; `epoch`, which epoch of the key it is; `granted_at`, when, as an
integer; and `roster` with its `roster_nonce`, or nothing. As its description says, the node
that hosts the channel relays a grant as an opaque blob and cannot decrypt it.

## `clipboard`

A record the person copied, as a structured snapshot: `source_schema` and `source_id`, the
schema of the record it was copied from and that record's id; `snapshot_json`, the snapshot,
as JSON text; and `copied_at`, when.

## `contacts`

A person and how to reach them: `display_name`, the name shown; and `dids`, `emails` and
`phones`, lists of the person's DIDs, email addresses and phone numbers, each empty when none is
known.

## `governance.record`

A governance record: one record file as a row, and the file is the wire format. Its fields are
`name`, `kind`, `title` and `body`, always present; `status`, `type`, `blocked_by`, `built`,
`authority`, `serves`, `security`, `opened` and `updated`, each a string or nothing; `region`,
`domains` and `evidence`, each a list of strings or nothing; `header_rest`, a list of `key` and
`value` pairs; and `form`, a `lead` or nothing, and a list of `lines`, each a `key`, a `pad` and
a `tail`.

## `message.v1`

An encrypted message envelope, from one peer to another or to a channel: `from_did`, the
sender's DID; `to_did`, the recipient's DID, or nothing; `channel_did`, the channel's DID, or
nothing; `thread_id`, the thread it belongs to; `body`; `nonce`; `ephemeral_pub` and `epoch`, or
nothing; `sent_at`, when, as an integer; and `type`. As its description says, the daemon stores
and relays opaque ciphertext, and decryption is always client-side.

## `messages`

A chat message in a thread: `sender_did`, who sent it, by DID; `body`; `thread_id`, the thread;
and `sent_at`, when.

## `model`

An installed model: its manifest, whether it is the active model of its kind, and when the
person installed it. `name`, `version`, `format`, `kind`, `parameters`, `quantisation`, `size`,
`context_window` and `template` describe the model; `licence` is its `spdx` identifier and a
`text_hash` of the licence's text; `publisher` is a string, or nothing; `weights` names the
weights, a sealed object, by hash; `active` is false unless this is the active model of its
kind; and `installed_at` is when.

## `notes`

A titled text note: `title`; `body`; `tags`, a list, empty when the note has none; and
`created_at` and `updated_at`, when it was written and last changed.

## `pin`

What one of the person's devices keeps of their drives: its default, or a drive or folder set
the other way. `device`, the device; `scope`; `drive` and `folder`, the drive or the folder set,
or nothing; and `value`, what is set.

## `tasks`

A completable work item, optionally nested: `title`; `due_at`, when it is due, or nothing;
`completed`, false until it is done; and `parent_task_id`, the task it is nested under, or
nothing.
