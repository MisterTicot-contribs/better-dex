[Introduction](README.md) • [Method](Method.md) • [Orders](Orders.md)
 • [Requirements](Requirements.md) • [Changes](Changes.md)
 • [Summary](Summary.md)

# Orders

## Market order

Cross existing offers, buy/sell immediately the requested quantity or fails.
Never register an offer in the orderbook: this is called *fill-or-kill*.

## Limit order

Cross existing offers if possible. The remaining quantity is registered as an
offer in the orderbook.

Optionally, limit orders can be made *post-only*; Which means that it will fails
if it crosses any order. This is useful against market moves when the purpose
of the offer is to provide liquidity. (or to avoid taker fees but this is
irrelevant on SDEX).

## Stop-loss order

This is an order that is executed when the price falls under a given threshold.
Traders are using it nearly systematically to cut positions that goes the wrong
way without further emotion/thinking.

Technically, this is a market order that is executed at the time the threshold
is met. It can't be registered in the orderbook beforehand.

## OCO order

OCO means "One Cancel Other". This is a limit order combined with a stop-loss
order. When taking a position, a trader often set one or several take-profit
levels, as well as a stop-loss. The exchange must then be able to execute this
strategy: this can be done with an OCO order.

Technically, if the stop-loss threshold is met, the limit order get canceled
and a market order is executed. If the limit offer is crossed, the stop-loss
order get canceled.

## Trailing-stop

A trailing-stop is a stop-loss order that gets automatically "moved" higher when
the price raises. It maintains a constant distance from the price, either
defined by an absolute value (a number) or a relative value (a percentage). This
is not as much used as stop-loss & OCO order, so I wouldn't say its an absolute
must-have, but it is a well-known and convenient trading tool. The reason I'm
mentioning it here is that, as we'll see later, it could be possible to
implement it on Stellar DEX at some point which is a really great thing.

## Combined orders

This one is not common at all. I even wonder if it really exist :). But what
matters is that it really makes sense in the context of the Stellar DEX.
Combined orders are orders that either all complete or all fail. The first use
that comes to mind is to combine two or more market orders.

Technically you need this to implement trading over pairs that have no volume.
For example if you want to buy JPY for EUR and there's no offer good enough you
would create a transaction with two orders: one market order that trade EUR for
XLM and one market order that trade XLM for JPY. You want both to execute fully,
or nothing at all. So it's a bit like pathPayment but for trades.

You can also handle arbitrage this way: Let's say two USD assets have their
prices diverging, and you want to sell the more expensive one for the cheapest.
This is a typical arbitrage operation: you make money while providing stability
to the market. If they don't have a shared orderbook you'll need to go through
XLM. However, you don't want to get stuck in the middle with XLM or having
bought one without selling the other. The transaction must either complete
entirely, or fail.
