# The ownership thesis

Kotton rests on one sentence: **the people who generate data are the people who own it.**
Everything else, the cryptography and the sandbox and the way the code is laid out, is there to
make that sentence true of a real person on a real machine. This essay says what the sentence
means, and why ownership, rather than something nearby, is the property worth building for.

## Ownership is not privacy

The two are easily confused, and the confusion costs a great deal.

Privacy is concealment. Concealment is useful, and Kotton uses it heavily: your things are
sealed before they leave your device, and even their names are blinded, so a host keeps uniform
blocks it cannot tell apart. But concealment on its own cannot be won, because there is always
one more observer. Every wall has a far side, and somebody stands there.

Ownership is about **value and standing**: who holds the thing, who can be paid for it, and who
must answer when it is demanded. A party can only be paid for what it holds. A holder can sell,
license, withhold, set a price or refuse; a party that holds nothing can do none of those, and
so has nothing to bring to a negotiation. That is how your data can be worth a great deal while
you are worth very little in the trade it feeds. No villain is required to explain it. You
arrived at the table with nothing to trade.

Move the holding and the arithmetic moves with it. When your things are kept under a key you
made, anyone who wants them has to ask you, and asking is the whole difference. Someone who can
be asked can refuse, agree on terms, agree with one party and not another, or agree this year
and decline the next. **Nobody else can be asked instead, because nobody else has it to give.**

That is the leverage, and it shows up in plain places. A demand for your records goes to you,
because you are the one who holds them; a landlord who keeps your sealed packs, asked for them,
can produce a byte count. The exception is a host that runs your daemon for you: that daemon
holds your keys on the host's machine, which is why a node's unproven posture is a named
[concession](../docs/design/concessions.md) and why
[the principles](../docs/design/principles.md) mark leaving such a host as not yet whole. No
broker sits in the middle, because there is no middle where your life is gathered to be sold.
And "we promise not to look" was never an answer. A promise is the holder describing its own
restraint; the ability to look stays where it was, and so does the holding. A key is not the
price of owning a thing. It is the owning.

What this buys is not a cheque. One person's data is worth very little, and a scheme that
promised you a payment for your exhaust would be promising pennies. What ownership buys is
standing, and **standing composes**. People who hold can act together: set common terms, refuse
together, license as a body. A group whose members have nothing to withhold cannot be organised
at all. The demand for what people produce is not going away; recommendation, research and every
appetite of the trade will want tomorrow what they take today. Whether they take it or buy it is
decided by where it sits.

## The design test

So the question to ask of any part of the system is not "can someone see this?" There is always
someone. The question is:

> **Does this compose into an asset, and whose?**

Data becomes someone else's asset when observations compose: when a durable identifier lets
today's observation join last month's into a record that can be sold, subpoenaed or built on. A
single observation is exhaust. A linked history is property. Find the join key and remove it.
That is usually a smaller and sharper change than trying to hide everything, and it is the one
that holds.

You can watch the test at work in what Kotton is. Your identity is a key you generated, with no
account behind it, so there is no table with your row in it and no issuer that learns every door
you approach. A relay that carries your traffic answers to a handle that is per contact and per
period, never a durable identity, so it holds nothing it could join to last month. One operator
key never operates two instances, because an observer who saw both could tell they were one
person's. A host keeping your store sees uniform blocks under blinded names and cannot tell
whether two of them are siblings.

The test also says where the work is not finished, and Kotton says so rather than leaving it to
be found. A host still sees how much you keep and when you touch it. One landlord who runs both
your relay and your vault can join what each learns, which is why that arrangement is shown to
each of your contacts. [The design](../docs/design/README.md) names these, and
[the concessions](../docs/design/concessions.md) give the way back from each.

## Said as value, not grievance

There are two ways to tell all of this, and only one of them is the claim. "You own it" is the
claim. "They can't see you" is the same fact told as grievance, and it only makes sense to a
reader who already feels wronged. So Kotton describes what it lets a person do: put their things
on any host, let software work on their records without handing them over, be asked rather than
taken from. Wherever both are true, "you can grant it" beats "nobody can see it".

This is not a matter of tone. A grievance is about the past and about somebody else; it asks the
reader to be angry and offers them an enemy. A value is about what the reader holds now and can
do next. The honest version of the argument does not need an enemy. What was built to keep
people's lives was built well, at great cost, when an ordinary person had no safe way to hold a
key and nothing to keep a life in that they could run. It is a solution that has outlived its
problem.

The same discipline keeps the argument from asking anyone to want less. Nothing here asks you to
give up what you like about the internet. You keep the feed, the search, the recommendations and
an assistant that knows your life. Much of what drives any recommendation was never watching in
the first place but publication: what people posted, praised and put their name beside, which is
public by construction. Aggregation still happens, because a recommendation is made from many
people. The difference is that the aggregator asks and can be refused, which is how pooling has
worked in every trade where the people who supplied a thing kept owning it. **The target is the
same internet, with the holding moved.**

Nor is the argument that anyone should leave what they use. It is that they should be able to. A
person shown a real alternative at an honest price, who weighs both and stays exactly where they
are, has done the thing this essay argues for. A payment is what you are offered instead of that
choice, never in compensation for it.

The costs are real, and they are stated. Most people will not run their own machine, and Kotton
must never require it; because a host keeps sealed blocks either way, keeping your things
somewhere else is a question of price and uptime rather than of whom you trust. Some of what the
ideal wants is still expensive or still research, and conceding to that is healthy. A concession
that quietly moved the holding back would not be a concession. And a market can be ugly:
standing includes the standing to make a bad trade. The alternative on offer is not a better
trade. It is no trade and no standing.

What it is all for is simple to state. **You become somebody who can be asked.** Sealed storage
and blinded names make the holding real rather than nominal. Confinement lets other people's
software be useful without becoming a holder, which is the subject of
[the confinement thesis](confinement-thesis.md). Grants make permission a thing you issue and
withdraw, which turns holding into standing. A self-issued identity removes the last party who
could be asked about you in your place.

Nobody pays for any of it yet, and nobody but the author has run it. The work is assembly rather
than discovery: every piece it needs already ships somewhere, built in different rooms for
different reasons. The platform is finished when a stranger can be asked, can refuse, and can be
paid, and when the machine they are speaking to can prove what it is unable to do.
