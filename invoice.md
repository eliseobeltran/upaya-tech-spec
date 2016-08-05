# Invoice API
We'll need an endpoint to create NetSuite `Invoices`. Depending on the answers to the questions below, we'll have a better understanding of whether or not update and delete functionality on these objects is required.

### Open Questions
**When Do We Create Invoices:**
We are working under the assumption that `Orders` in Guidebook which map to `Invoices` in NetSuite, will originate from Guidebook and be passed to NS in order for NS to have record of this and to send the invoice to the customer.

I think we're still fuzzy on _when_ this should happen. Is Guidebook responsible for sending emails to the customer during the quote negotiation process?

Do we need to use NetSuite at all for emailing the customer? Or can Guidebook take care of all customer communication between and only create `Invoices` _after_ the `Order` has been finalized and signed off by both parties?

**Payment Frequency:**
Will we need to create `Invoice` objects in NetSuite every time we require payment from a customer?

For example, if a customer purchases 1 Guide and opts to pay quarterly, do we need to create 4 `Invoice` objects in NetSuite? Or can we simply create a single `Invoice` and denote the payment frequency within it?


-----------

### Invoice Creation
This endpoint will allow us to create new `Invoice` objects in NetSuite. Ideally, upon successful `Invoice` creation, NS will return the `pk` of the Invoice in NS so Guidebook can persist it on our end for future calls.

**URL Pattern:** `guidebook-prod.netsuite.com/invoice/`

**Method:** `POST`

**Notes:** Note that `payment_due_date` is a `DateTimeField` and we persist all of our date times in UTC.

**POST Request Data:**
```json
{
    "customer_id": 12345,
    "items": [
        {
            "product_id": 1,
            "quantity": 5,
            "region_and_currency": "US:USD",
            "price_per_product": 500,
            "payment_due_date": "2015-06-02:00:00:00 - UTC",
            "term_start_date": "2015-05-02:00:00:00 - UTC",
            "term_end_date": "2016-05-02:00:00:00 - UTC"
        }
    ],
    "account_executive_user_id": 123,
    "account_manager_user_id": 125,
    "sales_development_rep_user_id": 129,
    "send_invoice_email": false,
}
```

**Expected Responses:**
  * `201 - Created`:
    * Description: A new Invoice was successfully created in NS.
    * Response:
    ```json
    {
        "invoice_id": 12345
    }
  ```
  * `400 - Bad Request`:
    * Description: NS could not create a new Invoice record b/c invalid or incomplete data was submitted.
    * Response:
    ```json
    {
        "errors": {
            "customer_id": "This Customer does not exist in NetSuite.",
            "items.0.quantity": "You cannot have a quantity less than 1.",
            "etc...": "...."
        }
    }
    ```


----------------

### Invoice Edit
Unsure if we want this functionality. We were initially working under the assumption that NetSuite was a pure Accounting system and we would only use it to record finalized invoices + records of payment.

It sounds like this may be drifting and we may want to use NetSuite for physically invoicing our customers (eg: sending emails) as well. If this is the case, the data _may_ flow like so:

1. A Salesperson creates an `Order` in Guidebook.
1. Guidebook pushing that data to NetSuite as an `Invoice`.
1. Customer rejecting the `Invoice` because they want to renegotiate.
1. Salesperson updates the existing `Order`, which is subsequently pushed to NetSuite and emailed to the `Customer`.

If this^^ is the case, we will need `Invoice` edit functionality.

That said, I think we would prefer to make `Invoices` immutable such that the flow worked like this:

1. A Salesperson creates an `Order` in Guidebook (which is in a "quote" state internally).
1. Guidebook emails the client the `Order` details.
1. The two sides negotiate.
1. The two sides come to an agreement; the `Order` is flipped to an "agreed" state.
1. Guidebook pushes the `Order` into NetSuite as an `Invoice`.
1. The `Invoice` is simply a source of financial record keeping in NetSuite; there is no need to update an existing `Invoice` object.

However, if we want to update existing `Invoices`, we will want an interface like so:

**URL Pattern:** `guidebook-prod.netsuite.com/invoice/<ns_invoice_id>/`

**Method:** `PATCH`

**Description:** Passing an existing NetSuite invoice id, this will allow Guidebook to update existing information about a `Invoice`.

**PATCH Request Data:**
```json
{
    "customer_id": 12345,
    "items": [
        {
            "product_id": 1,
            "quantity": 5,
            "region_and_currency": "US:USD",
            "price_per_product": 500,
            "payment_due_date": "2015-06-02:00:00:00 - UTC",
            "term_start_date": "2015-05-02:00:00:00 - UTC",
            "term_end_date": "2016-05-02:00:00:00 - UTC"
        }
    ],
    "account_executive_user_id": 123,
    "account_manager_user_id": 125,
    "sales_development_rep_user_id": 129,
    "send_invoice_email": false,
}
```

**Expected Responses:**
* `200 - OK`:
  * Description: An existing Invoice was successfully updated in NS.
  * Response:
    ```json
    {
        "invoice_id": 12345
    }
    ```
* `400 - Bad Request`:
  * Description: NS could not update the Invoice record b/c invalid or incomplete data was submitted.
  * Response:
    ```json
    {
        "errors": {
            "etc...": "...."
        }
    }
    ```
* `404 - Not Found`:
  * Description: NS could not find the specified `Invoice` record.
  * Response:
  ```json
  {}
  ```

------------

### Invoice Delete
I'm skeptical that we need this functionality.


------------

### Invoice Read
Provides an interface to read a specific `Invoice` from NetSuite. I'm skeptical we'll need this if the idea is to have Guidebook be the source of truth that pushes data into these external systems.

**URL Pattern:** `guidebook-prod.netsuite.com/invoice/<ns_invoice_id>/`

**Method:** `GET`

**Description:** Passing an existing NetSuite invoice id, this will allow Guidebook to update existing information about a `Invoice`.

**GET Request Data:**
```json
{}
```

**Expected Responses:**
* `200 - OK`:
  * Description: An existing Invoice was successfully fetched in NS.
  * Response:
    ```json
    {
        "customer_id": 12345,
        "items": [
            {
                "product_id": 1,
                "quantity": 5,
                "region_and_currency": "US:USD",
                "price_per_product": 500,
                "payment_due_date": "2015-06-02:00:00:00 - UTC",
                "term_start_date": "2015-05-02:00:00:00 - UTC",
                "term_end_date": "2016-05-02:00:00:00 - UTC"
            }
        ],
        "account_executive_user_id": 123,
        "account_manager_user_id": 125,
        "sales_development_rep_user_id": 129,
        "send_invoice_email": false,
    }
    ```
* `404 - Not Found`:
  * Description: NS could not find the specified `Invoice` record.
  * Response:
  ```json
  {}
  ```