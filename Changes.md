[Introduction](README.md) • [Method](Method.md) • [Orders](Orders.md)
 • [Requirements](Requirements.md) • [Changes](Changes.md)
 • [Summary](Summary.md)

# Changes

To start with, here's what we have right now:

* **manageOffer:** take-or-make / selling / break
* **manageBuyOffer** (CAP-0006): take-or-make / buying / break
* **pathPayment** to oneself: fill-or-kill / buying / break
* **createPassiveOffer:** take-or-make / selling / break

Now if we make createPassiveOffer post-only (which would clearly be meaningful)
and add the other direction for pathPayment (pathDonation?) we have it all. The
break/skip flag could be handled better at transaction-wide level as it appears
to be more a generic scalability feature than something specific to orders.

## Make createPassiveOffer post-only

It would now fail if it crosses any offer at registration time. This is right
because createPassiveOffer is meant to be used to provide liquidity, so we don't
loose support for any use case in the move. In fact this is weird that currently
createPassiveOffer can match offers at registration time.

As post-only offers direction doesn't matter it's all we need on that side.

## pathDonation

This is to pathPayment what CAP-0006 is to manageOffer. This is needed because
fill-or-kill doesn't behave the same way depending on the direction:

pathPayment to oneself: Buy X assetA for a maximum cost of Y assetB
pathDonation to oneself: Sell Y assetB for a minimum value of X assetA

In the first case X is fixed and Y is capped, in the second one this is the
contrary.

**Note:** if CAP-0006 ends up being merge into manageOffer, this one should then
be merged into pathPayment.

## break/skip flag as a generic scalability feature

As you know Stellar transaction are atomic, meaning it either all succeed or all
fail. As I said, the ability to pack operations together is critical to the
network output. Hence, better favor it.

While atomic transactions is a perfectly good & meaningful default behavior,
there are cases were we know operations doesn't depends on each other. When
packing operations together, we don't want the whole bundle to fail if one
doesn't pass.

I already presented how this is relevant to order passing, but I feel that it is
more a general concept that could be applied in other situations:

* What happens in case an inflation pools redistribute lumens and one user merge
  its account on the same iteration ? (could be an exchange sending withdrawal
  as well)
* What happens in case there's an airdrop and one user close its trustline on
  the same ledger iterations
* What happens in case an user wants to close multiple orders at once but one
  got executed meanwhile?

While failures may be rare, the consequence could be tricky when we're talking
about automated systems. A non-atomic-transaction parameter would solve those
issues, providing the system is able to send a meaningful feedback about what
passed and what failed.

## Alternative design

Another possible design would be to implement everything ontop of manageOffer
algorithm as I first wanted, but the thing is pathPayment actually offer better
fill-or-kill capabilities than manageOffer possibly would. (I can develop that
point if needed)
