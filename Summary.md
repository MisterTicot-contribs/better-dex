[Introduction](README.md) • [Method](Method.md) • [Orders](Orders.md)
 • [Requirements](Requirements.md) • [Changes](Changes.md)
 • [Summary](Summary.md)

# Summary

The big picture is that we can cover all required functionalities with 5
operations:

* **take-or-make:** manageOffer, manageBuyOffer
* **post-only:** createPassiveOffer
* **fill-or-kill:** pathPayment, pathDonation

We can also merge manageBuyOffer into manageOffer as a switch, and pathDonation
into pathPayment as a switch as well; we would end up with three operations.

The steps bellow can be implemented either independently or at once, and in any
order.

## 1 - Go ahead with CAP-0006, or merge it into manageOffer

take-or-make orders have two direction: buy & sell. manageOffer is sell
direction and CAP-0006 implement buy direction.

## 2 - Implement pathDonation, or merge it into pathPayment

As for take-or-make, fill-or-kill is a directional operation. pathPayment
implement the buy direction and we need a new operation/switch that give us the
sell direction.

## 3 - Make createPassiveOffer post-only

We need post-only at protocol level because it can't be properly emulated.
createPassiveOffer is meant to be used to provide liquidity, so it's an
anti-feature that it can cross offers. This is the perfect candidate to carry
post-only functionality.


## 4 - Non-atomic transactions (transaction-wide parameter)

This is not directly related to the DEX, but this is a requirement so we can
pack orders together with having a singular operation failure cause rejection of
the whole bundle. This is a scalability feature that also makes sense in the
context of bundling payments.

## Optionally 5 - Work on immediate-or-cancel orders

I left it aside as a non-required order type. However it can't be emulated
properly and it could be cheaply implemented on top of manageOffers or
pathPayments. Having it implemented won't hurt.
