[Introduction](README.md) • [Method](Method.md) • [Orders](Orders.md)
 • [Requirements](Requirements.md) • [Changes](Changes.md)
 • [Summary](Summary.md)

# Requirements

Here's the minimal set of switches that I think is needed at protocol level to
make it possible to implement the listed orders at UX level. My rationale is
simple: if it can't be done at software level, but it is a must-have, then it
must be done at protocol level.

Those switches allows for 12 permutations, 10 of them only being meaningful (2
others being redundant). I'm not going to discuss whether it should all be
implemented into one operation or into 8 or something else. This is not
relevant to this part of the design.

I'm talking about switches because each of them could be implemented as a tiny
variation of the existing offer matching algorithm.


## Order type: take-or-make / post-only / fill-or-kill

* **take-or-make:** This is current the behavior of passiveOffer, manageOffer
  and manageBuyOffer: it trades against matching offers if any, then register an
  offer in the orderbook if the trade were not completely filled.
* **post-only:** This won't match any offer. If there's crossing offers, it
  rejects. As I said there's generally two reasons to use this kind of order:
  provide liquidity or avoid maker fees. The second one is irrelevant, so I
  think this should have the same property than passiveOffer matching-wise - and
  replace it as it would be a better option to provide liquidity.
* **fill-or-kill:** Fill completely under the given price or doesn't buy
  anything. Never register offers in the orderbook.

**Rationale:**

Those three types of orders are the basic elements on which the various trading
practices take places.

fill-or-kill and post-only behaviors can't be properly emulated using
take-or-make. The reason is, you can't know what will be the state of the
orderbook at the time your order will be validated. Emulation will work *most of
the time*, but edge cases can be nasty.

That single fact is actually the crux of the issue we're facing:

* A liquidity provider try to emulate post-only using take-or-make but, before
  it got validated, the orderbook moved and it ends up matching a counteroffer,
  making him loose money and taking volume out. (the exact opposite of the
  intended operation)
* A trader try to emulate fill-or-kill on a thin orderbook using take-or-make
  with a price of 0 but, before it get executed, some other big player cross the
  offer she'd expected to match. The offer ends up registering a limit offer at
  price 0. (have a nice day!)

Those edge cases shows current implementation limit. Explicit implementation of
those fundamentals orders types can fix that at a cheap cost.


## Order direction: buy / sell

* **sell:** This is the current manageOffer behavior: offer is capped by
  sellingAmount.
* **buy:** This is the behavior implemented by CAP-0006: offer is capped by
  buyingAmount.

**Note:** Order direction doesn't have effect on post-only because the price of
the trade is defined by the maker; hence both amountBuying & amountSelling are
fixed. (There's a redundancy, but no incompatibility)

**Rationale:**

See CAP-006 as it's the whole point of this CAP.


## Behavior on failure: break / skip

* **break:** This is the default behavior for Stellar operations: if an
  operation doesn't succeed the transaction is rejected.
* **skip:** The operation fails quietly, the transaction is still valid.

**Note:** This is about post-only and fill-or-kill rejecting. It may or may not
be extended to other possible errors. It could also be a transaction-wide
property. In fact I tend to think it could be a transaction-wide parameter.
Anyway I'll explore it in the context of passing orders for now.

**Rationale:**

The behavior on rejected post-only/fill-or-kill offers needs to be defined. The
current standard in Stellar is to reject the whole transaction. Another possible
behavior is to skip the operation, proceeding with the other ones. The case is
that both behavior are actually meaningful, hence the need for this switch.

Rationale for breaking: The obvious one is to stay consistent with how
operations behaves. Besides, in Stellar transactions, an operation may depend on
the success of the previous one. I already gave two examples when I described
combined orders. More generally, this can play a role into smart contracts.

Rationale for skipping: There's use case were you'd pack a lot of orders
together without them depending on each other. For example, a liquidity provider
could pass post-only offers at various prices; If some fails it doesn't
compromises the others. Another example: an exchange UI may want to pass market
orders on various unrelated orderbooks at once.

Rationale for keeping both: While the Stellar network output is good in term of
operations/seconds, it is still limited in term of transaction/seconds. Hence,
an important factor of scalability is the ability to squeeze multiple operations
into one transaction. The skip switch allow to pack together various operation
whose success doesn't depends on each other. I'm not sure about the actual
implementation, but I suspect this switch to be rather costless. Costless and
useful.
