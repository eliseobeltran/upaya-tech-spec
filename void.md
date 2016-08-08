# Payment > Void API
We will need to figure out what type of endpoint we want to implement in order to handle void requests from Guidebook.

-----------

### Definition
A voided `Order` is the cancellation of an `Order` after it has been approved, but before payment has been made.

In this case, an `Invoice` would already have been created in NetSuite which needs to be nullified.

-------------

### Guidebook Data + Flow
On the back-end, Guidebook stores the following data when an item is purchased:

`Order` => `Item` => `ItemPayment`

When a void is issued, we create a new, negative `ItemPayment` and attach it to the `Order`.

From there, we need make a call to NetSuite in order to denote that all items associated to an order were voided and that NetSuite should act accordingly.
