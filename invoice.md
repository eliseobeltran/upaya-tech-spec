# Invoice API
We'll need an endpoint to create NetSuite `Invoices`. We are thinking of NetSuite `Invoices` as immutable objects that are created _after_ a corresponding Guidebook `Order` has been "signed off" by both the customer and Guidebook.

-----------

### Data Flow
We are envisioning a data flow from Guidebook into NetSuite like so:

1. A Salesperson creates an `Order` in Guidebook (which is in a "quote" state).
1. Guidebook emails the client the quote details.
1. Optional negotiation.
1. Both sides come to an agreement, Sales Ops approves and the `Order` is flipped to an "accepted" state.
1. Guidebook pushes the `Order` into NetSuite as an `Invoice`.
1. The `Invoice` is simply a source of financial record keeping in NetSuite; there should not be a need to update any existing `Invoice` object.

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
If `Invoice` objects in NetSuite are immutable, there shouldn't be a need for this endpoint. However,if for some reason we do find the need to update existing `Invoices`, we will want an interface like so:

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

That said, here is the what we would expect that endpoint to look like.

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