[Introduction](README.md) • [Method](Method.md) • [Orders](Orders.md)
 • [Requirements](Requirements.md) • [Changes](Changes.md)
 • [Summary](Summary.md)

# Method

Implementing new operations while issues are raised doesn't seems a great way to
go, as we'll end up with half a dozen of them at least before a proper trading
environment is achieved. The reason is a couple of options are really needed:
the bare minimum being *post-only* and *fill-or-kill*. We can't afford making
new operations each time because the number of them would grow quadratically
(that is, out of control). I think that the right way to deal with it is
figuring it out and designing it well from the start.

So we should list the common orders that are available on any decent trading
platform. From there, we should work on a couple of operations that would cover
the most important ones thanks to cleverly designed switches. Then, we should
implement it all at once so we limit the number of deprecated operations.
