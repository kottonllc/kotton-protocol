# Concessions

Cryptography is not all caught up, most people will not self-host, and a first release has to
exist. So concessions are made, on one rule: each is explicit, each leaves a seam back toward
the ideal, and each leaves Kotton the most protective option in its class even in conceded
form. Convenience, performance, decentralisation and timeline are conceded. A user's
structural ownership of their data is not.

| Concession | Today | The seam back |
| --- | --- | --- |
| A relay a person rents and stands at is handed which relays their device reaches, beside the address the device connects from (RFC 0007) | addresses, times and sizes, and of each open it carries a sealed frame of one length; no handle of a door at another relay, no content, no identity, nothing it can open. What lasts is the device's address and the far relays' addresses and keys. If its operator keeps it, it is a record at that operator: principle VII rests there on the operator and on the person's choices | asking no one: remove the relay, having added another; run a relay and stand at it alone; or put a network of your own choosing beneath, so the address handed over is that network's. Adding a relay is not by itself a way back: each relay in a set is handed the same. Not designed: padding and timing across relays. No seam is sought in the protocol for the far relays' addresses: a party between is the person's to put beneath |
| A carried open is sealed under a value of the far relay that the relay carrying it reports (RFC 0007) | reported truly, nothing a carrying relay kept is opened by the far relay's key, should that key later be taken; reported otherwise, that open was refused, and the relay that reported would read its two handles with that key. At first contact the key sealed to is the carrying relay's word too, for handles made for one ceremony | not built: a channel of the device's own to the far relay, made through the relay that carries, in which the far relay proves its fresh value to the device, at an exchange more on every open |
| A relay one person alone stands at is that person's address at every relay their contacts stand at (RFC 0007) | one lasting address, the relay's and no device's; a relay kept at home is the home's | stand at a relay others stand at too, which asks whoever runs it, and remove the relay you stand at alone, since a delivery goes by every relay you stand at |
| The phone is the browser | a phone reaches Kotton through its browser, which is trusted as far as its vendor and its extensions are | Kotton installed on every machine a person uses, and then a bare-metal capability operating system, where the same rule holds from the hardware upward |
| A node's posture is asserted, not proven | no attestation; a daemon a host runs for you holds your keys on the host's machine | hardware attestation, and past it, computation on sealed data |
| A host sees how much you keep and when you touch it | volume and access pattern, and no plaintext | fixed allocation and oblivious access, the known answers, not yet built |
| Those admitted to a relay share a key (RFC 0007) | a key says one of those who hold it and never which, so a relay keeps no row for a person; one person is removed by rotating the key for all who held it | not designed: a token of admission that proves its holder is one of those admitted and cannot be linked to another |
| A vault's door for the hour is a label whoever holds the vault's line can make (RFC 0007) | a relay admits to that door any such holder its rule lets stand; what answers a knock is proved by the vault's key and nothing else | not designed: a label only the vault can stand under, a key blinded for each hour |
| The network a device is on sees how many doors it holds (RFC 0007) | a count, and when it changes; no label, key or address | not built: a reservation that carries many labels on one connection |
| A device set to dial a vault directly cannot do so over an onion route (RFC 0007) | that path is QUIC, which is datagrams; the path through a relay is a stream and runs over any network | the stream RFC 0005 names for a vault that cannot accept QUIC. A relay that answers only on an onion route is carried to by no other relay; its person adds another to be reached at |

A concession that is not on this page is not a concession. It is a bug.
