# The confinement thesis

[The ownership thesis](ownership-thesis.md) says who should hold your things: you. This essay is
about how that survives use. Software has to be able to work on your records, and the moment it
does, it is in a position to become a holder itself. Confinement is how Kotton lets it do the
work without ever holding the thing.

Computers have tried two arrangements. The desktop trusts everything: you and every program you
run share the same authority, so you are free, and one bad install is a stranger with your
standing. The phone trusts nothing: every program is walled into its own cell and a gatekeeper
decides what may be installed at all, so you are safe, and caged alongside the programs. Both
are answers to one question, *is this software trustworthy?*, and that question cannot be
answered in advance by you, by a reviewer, or by anybody. A review can establish that a program
declares its intentions. It cannot establish that the program keeps them.

Kotton asks a different question: **what can this program do if it is not trustworthy?** Make
the answer "very little" and it stops mattering who wrote it. So the rule is: **confine the
applications, trust the person.** You decide what to run and what to grant, with the freedom of
a desktop. What an application can do with what you granted is decided by its confinement, and
not by its manners. That is why there is no registry and no review queue to pass: the cell
protects you, not a guard at the door.

## A process holds exactly the authority it acts on

An application in Kotton is a WebAssembly component, and it can call only the imports its
contract offers. There is no network import, and no file import that takes a path. What there
is: reading and writing inside a container you granted, a surface to draw on, a paste you made,
delivered by the compositor, and a way to ask for more, which puts a consent prompt in front of
you. That prompt is drawn by the compositor, which an application cannot draw over. A grant
names a container, never a path, so an application given your photos cannot infer the shape of
the rest of your tree. Every call passes one check in the daemon that walks the grant against
what the call asks for. The software this project writes is confined on exactly the same terms;
there is no first-party tier.

A program that can read everything and reach nothing is very nearly harmless, so **reading can
be cheap, and the exit is the one thing to watch.** Other platforms fence each entrance
separately, with questions nobody can evaluate about a program they have not used yet. Kotton
watches the exit instead. No application reaches the network;
[the principles](../docs/design/principles.md) say so of every app. What an application may send
goes only to a person, as a message, a file or a call, only with your leave, through the daemon,
which seals it and carries it. A photo sorter must read your photos; without such a grant it has
nowhere to take them. Revoking a grant ends the next call; what was copied before that is not
reached, which is why applications work on your records in place rather than keep their own.

The same rule lays out Kotton's own processes. Your keys are held by the daemon that acts for
you, and they do not cross into anything else: the compositor, the inference runtime, the
process that watches your files for sync, and the gateways that accept connections from the
network all hold none. In the design, one process, the device daemon, reads users' roots from
the platform's keychain and hands each user's daemon its own, so no process that acts for you
opens the keychain for them; today the daemon that acts for you reads its roots there itself,
and the single reader is on [the list of what is not built](../docs/design/not-yet.md). A
language model is a large, untrusted input to a large runtime, so the process that parses one
can open neither your store nor the network. An application is given views of your things, never
keys to them.

Two limits belong here. *Nearly* harmless is doing real work in that sentence: a confined
program can still show you misleading things, or waste your attention. And confinement bounds a
program; it does not improve it. You will still have to choose software well. You will simply
survive choosing badly. Where the confinement is not yet finished,
[the design](../docs/design/apps.md) says so. The signed record of what you approved at install
is not yet what the runtime reads, so an application's declared default is its grant. Sending a
message is linked today only for the app you designate as your shell, and whether any other may
is not decided; a call can be asked for, and nothing carries one yet. The airlock, a holding
space for bringing things in, is not wired.

Confinement is also why somebody else can run your infrastructure without governing it: a host
is arranged the same way, given the work and not the keys. Your things are sealed on your device
before they go anywhere. [A vault](../docs/design/nodes.md) you rent keeps sealed packs it
cannot open; it has no verb that returns plaintext, and a build check refuses one. Asked for
your records, its landlord can produce a byte count. A relay joins two of your devices when they
cannot reach each other and holds nothing past delivery. So hosting becomes something you buy
for uptime, and leaving is your own act: admit a second vault, copy to it, destroy the first.
The first landlord is never asked.

## Why it matters most for an assistant

For forty years, computing has been kept safe by a careful person in the loop, the one who does
not open the attachment. An assistant that acts on your behalf takes that person out, because
asking about every step is what made it useless. And to a language model, an instruction and a
description are the same kind of thing: a sentence in a web page telling it to send your private
folder somewhere arrives exactly like a sentence from you.

So the assistant Kotton is designed around holds no authority at all. It reads broadly, because
that is its use, inside a runtime with no key, no store handle and no network. It proposes; you
dispose: what it puts in front of you is an ordinary action that you take or do not. Today
nothing grants it any action of its own: a standing policy, set by you for each kind of action,
under which what is reversible and stays with you could go ahead without asking, is not built.
And an application that may ask the local model holds no way to send a message or place a call,
so a hostile sentence in what it read has no path to anyone else: what it can still reach is
what you granted it, and no more.

**The assistant does not run yet.** The confined inference runtime exists, runs on Linux only,
and cannot yet load a real model; the assistant itself, the index it would read, and the trusted
path it depends on do not exist. [Not yet](../docs/design/not-yet.md) says so: what is said here
is its design.

## The staged path, and what is conceded

Today Kotton is daemons on Windows, macOS and Linux, a runtime on top of an operating system it
did not write, so the machine underneath is trusted the way any operating system is. That is a
limit of running on today's platforms. On Linux a kernel profile adds a second, wider bound
around the code that runs applications; on macOS and Windows that second bound is not built yet,
and the runtime's own resolution is the only one, as [apps](../docs/design/apps.md) says.

A phone reaches Kotton through its browser, which is trusted as far as its vendor and its
extensions are. That is a concession, made on one rule: it is named, it leaves a seam back
toward the ideal, and it leaves Kotton the most protective option in its class even in conceded
form. The browser on the phone gives way to Kotton installed on every machine a person uses, and
that gives way to the end of the road: a bare-metal capability operating system, where the same
rule holds from the hardware upward and nothing beneath you has to be trusted at all. Two more
concessions are made on the same rule. A node's posture is asserted today, not proven; the seam
is hardware attestation, and past it, computation on sealed data. A host sees how much you keep
and when you touch it; fixed allocation and oblivious access are the known answers, not yet
built.

One limit is sharper than these. A host that runs your daemon for you can decline to start it,
which leaves you keys to ciphertext you must move before you can reach it. That is the one
principle Kotton does not yet keep, and [the principles](../docs/design/principles.md) mark it
false rather than quiet.

What is conceded is convenience, performance, decentralisation and time. **What is never
conceded is that you hold the keys.** [The concessions](../docs/design/concessions.md) list each
concession with its way back. A concession that is not on that page is not a concession. It is a
bug.
