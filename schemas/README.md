# Schemas

The record schemas the first release's decision (RFC 0003) names: the keychain, the
whole-keeper claim RFC 0002 defines, and the access log. Each file here is the authority for
its schema, in the form a schema is registered in: its NSID, its version, its fields with their
types, and a description. An implementation holds these in its core set, unchanged; the
reference holds them in `kotton_schemas`'s core set. A change to a field is a new version of
the schema.

Every record is sealed under its person's key, in a drive or on a device, and a vault or a
relay reads none of it. No field below is an identifier minted for the record, and a record's
own id is the implementation's.

## `sh.kotton.keeps-whole`

That one device keeps one drive whole, as RFC 0002's types say: `device`, the identifier of the
key the device presents, and `drive`, the drive's id. That the record exists is the claim. It
is written in the drive it names, by that device alone, and deleted when the device stops
keeping the drive whole.

## `sh.kotton.secret`

A password or a token in the person's keychain, behind `secret`: `label`, what the person calls
it; `value`, the secret itself; `kind`, `password` or `token`; and `note`, anything else the
person writes beside it, or nothing. No app reaches it: a schema grant on `sh.kotton.secret` is
refused, and it is kept out of record search.

## `sh.kotton.access-log`

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
- `what`: what was read or written, as the person names it: a drive path
  (`<drive>:/<path>`) or a record (`kotton://records/<nsid>/<id>`). It is never a digest of
  the content. `<drive>` is the drive's name as the person sees it, not its id; a renamed
  drive's earlier entries keep the old name.

No app reaches it: a schema grant on `sh.kotton.access-log` is refused, as on
`sh.kotton.secret`, and it is kept out of record search, so only the person's own CLI or shell
reads it.
