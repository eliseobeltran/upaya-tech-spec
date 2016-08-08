# Payment > Refund API
We will need to figure out what type of endpoint we want to implement in order to handle refund requests from Guidebook.

-----------

### Definition

A refund is a negative payment back to the customer. In the context of Guidebook, refunds typically occur when an unsatisfied customer (whom we want to keep), gets partial money back against a previous `Order` (more specifically, on our end against a previous `ItemPayment`).

In practice, we refund money to the customer and allow them to continue to use whatever items were refunded. For instance, if the purchased a Plus guide and their event stunk and they requested a refund, we would review their request, refund them some or all of their payment and allow them to continue to use the guide.

-------------

### Guidebook Data + Flow

On the back-end, Guidebook stores the following data when an item is purchased:

`Order` => `Item` => `ItemPayment`

When a refund is issued, we create a new, negative `ItemPayment` and attach it to the `Order`.

From there, we need make a call to NetSuite in order to denote that the an item was refunded.

