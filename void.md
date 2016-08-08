# Payment > Void API
We will need to figure out what type of endpoint we want to implement in order to handle void requests from Guidebook.

-----------

### Definition

A void payment is a negative payment back to the customer, then a set of matching positive payments that must be made by the customer to Guidebook. In the context of Guidebook, voids typically occur when an `Order` was created to an incorrect bill-to address.

Since we are building a fairly robust `Order` state system (ie: Orders will be reviewed by multiple parties in a "quote" state until they are approved by all parties and finally kicked over to NetSuite), I doubt we'll see voided Orders frequently.

-------------

### Guidebook Data + Flow

On the back-end, Guidebook stores the following data when an item is purchased:

`Order` => `Item` => `ItemPayment`

When a void is issued, we create a new, negative `ItemPayment` and attach it to the `Order`. We then create another, new set of positive, pending `ItemPayment` records to denote that the customer has a balance due.

From there, we need make a call to NetSuite in order to denote that the an item was voided and new payments have been created.
