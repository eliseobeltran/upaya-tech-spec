# Upaya Tech Spec
This repository outlines the general communication flow, request/response cycle and data format that Guidebook expects to use to communicate with NetSuite (via Upaya).

## Model Mapping (and Questions)
This is our understanding of the models in SalesForce, Guidebook and NetSuite and the relation + mapping between each of them.

|SalesForce|Guidebook|NetSuite|Notes|
|----|----|----|----|
|Customer|Account|Customer|Guidebook does not have a parent `Customer` model.|
|Contact| ... |Contact|It is my understanding that both SFDC + NS have `Contacts` that get rolled up into a parent `Customer` object. It is _not_ necessary for Guidebook to track these users. Is it 100% necessary for these objects to get passed into NS from SFDC? If so, why?|
|Opportunity > Quote|Order|Invoice|Internal question to understand if we need to store SFDC `quote_id` on Guidebook's `Order` model.|
||Payment|Payment?|Guidebook will track individual payments per order and per payment frequency. It seems necessary to send this data to NS for rev rec; what is the model called in NS?|

## API Interface
Below are the NetSuite endpoints we believe we'll need in order to send data from Guidebook into NetSuite. Note that these are simply proposals and fully expect feedback from Upaya before we land on an agreement.

#### Data Format
We'd prefer to use JSON as the communication language for both requests and responses.

#### Endpoints
Below are the endpoints we'll need to come to agreement on. Note that I'm excluding `401 - Unauthorized` and `403 - Forbidden` status codes in these documents; we're working under the assumption that those will be implicitly handled by whatever token + secret key auth scheme that NetSuite provides.

* [Customers](customer.md)
* Contacts (TBD)
* [Products](product.md)
* [Invoices](invoice.md)
* [Payments](payment.md)
  * [Refunds](refund.md)
  * [Voided Orders](void.md)
* Constants
  * [Business Unit Choices](biz-units.md)
  * [Industry Choices](industries.md)
