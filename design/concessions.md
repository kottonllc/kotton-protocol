# Concessions

Cryptography is not all caught up, most people will not self-host, and a first release has to
exist. So concessions are made, on one rule: each is explicit, each leaves a seam back toward
the ideal, and each leaves Kotton the most protective option in its class even in conceded
form. Convenience, performance, decentralisation and timeline are conceded. A user's
structural ownership of their data is not.

| Concession | Today | The seam back |
| --- | --- | --- |
| A rented relay learns which relays its tenant reaches (RFC 0007) | addresses, times and sizes, and the handles of each open for their hour; no identity, nothing that lasts past its hour but a relay's address, nothing it can open | asking no one: run the relay yourself, or leave by standing at another relay and signing the reach note again. As a convenience, which asks whoever runs it: stand at a relay someone you trust runs. Not built: a second hop, so that no one relay sees both ends, and an open sealed to the far relay, so the carrying relay passes on a door it cannot read |
| A relay one person runs alone is that person's address at every relay they reach (RFC 0007) | one durable address, the relay's and no device's; a relay kept at home is the home's | stand at a relay others stand at too, which asks whoever runs it. Not built: a second hop |
| The phone is the browser | a phone reaches Kotton through its browser, which is trusted as far as its vendor and its extensions are | Kotton installed on every machine a person uses, and then a bare-metal capability operating system, where the same rule holds from the hardware upward |
| A node's posture is asserted, not proven | no attestation; a daemon a host runs for you holds your keys on the host's machine | hardware attestation, and past it, computation on sealed data |
| A host sees how much you keep and when you touch it | volume and access pattern, and no plaintext | fixed allocation and oblivious access, the known answers, not yet built |
| Those admitted to a relay share a key (RFC 0007) | a key says one of those who hold it and never which, so a relay keeps no row for a person; one person is removed by rotating the key for all who held it | not designed: a token of admission that proves its holder is one of those admitted and cannot be linked to another |
| A vault's door for the hour is a label whoever holds the vault's line can make (RFC 0007) | a relay admits to that door any such holder its rule lets stand; what answers a knock is proved by the vault's key and nothing else | not designed: a label only the vault can stand under, a key blinded for each hour |

A concession that is not on this page is not a concession. It is a bug.
