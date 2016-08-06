# Payment API
We'll need an endpoint to create NetSuite `Payments`. We are thinking of NetSuite `Payments` as immutable objects that are created _after_ a corresponding Guidebook `ItemPayment` has been updated with valid payment information.

-----------

### Data Flow
We are envisioning a data flow from Guidebook into NetSuite like so:

1. A Guidebook `ItemPayment` (which records payments due as well as payments made) is created when an `Order` is finally accepted. It has a `payment_due_date` as well as a `payment_received_date` (as well as other payment meta data).
1. Once payment has been received, an `ItemPayment` has its payment meta data updated and from then on, it is immutable.
1. Once the `ItemPayment` has been updated w/ the relevant payment info and goes into the state described above, a call is made to NetSuite to create a `Payment` object. From our understanding, this object is also immutable and serves as record of payments received for financial reporting.

-----------

### Payment Creation
This endpoint will allow us to create new `Payment` objects in NetSuite. Ideally, upon successful `Payment` creation, NS will return the `pk` of the Payment in NS so Guidebook can persist it on our end for future calls.

**URL Pattern:** `guidebook-prod.netsuite.com/payment/`

**Method:** `POST`

**Notes:** Note all of our date related attributes on the `ItemPayment` model are represented by `DateTimeFields` and stored in the UTC timezone.

**POST Request Data:**
```json
{
    "invoice_id": 12345,
    # customer is inferred by invoice.customer
    "currency": "US:USD",
    "rack_price": 300,
    "actual_price": 200,
    "purchased_product_id": 1,
    "payment_processor_id": 1,
    "payment_processor_payment_id": "hashed-code",
    "payment_due_date": "2015-05-02:00:00:00 - UTC",
    "payment_received_date": "2015-02-02:00:00:00 - UTC",
}
```

**Expected Responses:**
* `201 - Created`:
  * Description: A new Payment was successfully created in NS.
  * Response:
  ```json
  {
      "payment_id": 12345
  }
  ```
* `400 - Bad Request`:
  * Description: NS could not create a new Invoice record b/c invalid or incomplete data was submitted.
  * Response:
  ```json
  {
      "errors": {
          "invoice_id": "This Invoice does not exist in NetSuite.",
          "etc...": "...."
      }
  }
  ```


----------------

### Invoice Edit
My understanding as that these objects are immutable. There should be no need to edit them.

------------

### Invoice Delete
I'm skeptical that we need this functionality.


------------

### Invoice Read
Provides an interface to read a specific `Payment` from NetSuite. I'm skeptical we'll need this if the idea is to have Guidebook be the source of truth that pushes data into these external systems.

That said, here is the what we would expect that endpoint to look like.

**URL Pattern:** `guidebook-prod.netsuite.com/payment/<ns_payment_id>/`

**Method:** `GET`

**Description:** Passing an existing NetSuite payment id, this will allow Guidebook to update existing information about a `Payment`.

**GET Request Data:**
```json
{}
```

**Expected Responses:**
* `200 - OK`:
  * Description: An existing Payment was successfully fetched in NS.
  * Response:
  ```json
  {
      "invoice_id": 12345,
      "currency": "US:USD",
      "rack_price": 300,
      "actual_price": 200,
      "purchased_product_id": 1,
      "payment_processor_id": 1,
      "payment_processor_payment_id": "hashed-code",
      "payment_due_date": "2015-05-02:00:00:00 - UTC",
      "payment_received_date": "2015-02-02:00:00:00 - UTC",
  }
  ```
* `404 - Not Found`:
  * Description: NS could not find the specified `Payment` record.
  * Response:
  ```json
  {}
  ```