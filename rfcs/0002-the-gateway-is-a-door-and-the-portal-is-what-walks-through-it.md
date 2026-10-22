# 0002: The gateway is a door, and the portal is what walks through it

## Context

RFC 0001's table names one repository, `kotton-web`, for "the browser and phone client, the
pairing page, the gateway". That puts two trust levels in one name, the same fault the table
avoids elsewhere by separating the compositor from the desktop. The gateway, `khttpd`, is a
process in the system's table: the HTTP and WebSocket door for a browser, holding a
remote-origin token and able to reach no local-tier verb. It decides what the network may ask.
The client that a person opens in a browser, on a phone or anywhere else, is an app that
speaks the protocol through that door, and anyone could write another.

The name "web" also named the delivery, not the thing, as "http" would have named the wire.

## Decision

The row for `kotton-web` is replaced by two:

| Repository | What it is |
|---|---|
| `kotton-gateway` | the door from the network: `khttpd`, the HTTP and WebSocket process a browser reaches; trust path, holding a remote-origin token and reaching no local-tier verb; one implementation among any that speak the gateway's surface of the protocol |
| `kotton-portal` | the page a person opens from any browser, on any device, to reach their things at home, and the pairing page; an app, one client among any |

Repositories are named for the surface or the role, never for the technology that delivers
them: terminal, desktop, portal; gateway, relay, vault. The rule binds repositories; a crate
inside one may still be named for what it does.

Nothing else in RFC 0001 changes. "Portal" names what the page is, a way through to a person's
things at home, and it would still be the right word if the page were served some other way.

## Consequences

- The system's process table gains nothing: `khttpd` was already a process. It gains a
  repository, cut when its rung comes, under the rule RFC 0001 gives.
- The portal is held to the protocol alone, like the desktop and the CLI: it may not need
  anything a stranger's client could not also have.
- The pairing page goes with the portal, not the gateway: it is something a person opens, and
  the door it walks through is the gateway's.

## Reaches

- The organisation's page, where the two repositories are listed.
- `kotton`, the system repository, which today holds the gateway process as `core/khttpd` and
  the page and its pairing page as `client/web` and `client/web-pair`. Those move to the two
  repositories when they are cut.
- The two repositories themselves, when they are cut.
- No protocol version: this RFC changes the names and boundaries of repositories, not the
  protocol. No request, type or encoding changes.
