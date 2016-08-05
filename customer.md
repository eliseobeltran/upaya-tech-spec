# Customer API
We'll need an endpoint to create, update (and delete?) NetSuite `Customers`.

### Open Questions
**Parent/Child Structure:**
It sounds like NetSuite supports a data structure in which `Customers` act as "parent" objects for `Contacts`. Guidebook doesn't necessarily support this direct relationship (we loosely model it with Orgs + User Accounts, but not exclusively for billing purposes). Is it necessary for NS to receive a set of `Contacts` before creating a new `Customer`; or vice-versa, does NS expect a list of `Contacts` when it creates a new `Customer`?

**Email Uniqueness:**
Is `email_address` a unique field on the `Customer` model in NetSuite? eg: Can NS have two `Customers` with the same email address? Note that `email` is a unique field on our `Account` model.

**SalesPerson Association:**
Does NetSuite need Salesperson details attached to `Customer` objects? ie: Do we need to send along our AccountExec, AccountMgr and SalesDevelopmentRep details when creating or updating `Customers` in NS?

-----------


### Customer Creation
This endpoint will allow us to create new `Customer` objects in NetSuite. Ideally, upon successful `Customer` creation, NS will return the `pk` of the Customer in NS so Guidebook can persist it on our end for future update + delete calls.

**URL Pattern:** `guidebook-prod.netsuite.com/customer/`

**Method:** `POST`

**POST Request Data:**
```json
{
    "first_name": "Foo",
    "last_name": "Bar",
    "email": "foo@bar.com",
    "address": "2345 Some Street",
    "city": "Palo Alto",
    "state": "CA",
    "zip": 94054,
    "business_unit": "EDU",  # choices==['EDU', 'CORP', 'ENTERPRISE']
    "guidebook_account_pk": 23423,

    # (Optionally)
    "contacts": [
        {"email": "contact_1@example.com"},
        {"email": "contact_2@example.com"},
    ]
}
```

**Expected Responses:**
  * `201 - Created`:
    * Description: A new Customer was successfully created in NS.
    * Response:
    ```json
    {
        "customer_id": 12345
    }
  ```
  * `400 - Bad Request`:
    * Description: NS could not create a new Customer record b/c invalid or incomplete data was submitted.
    * Response:
    ```json
    {
        "errors": {
            "first_name": "Invalid customer first name.",
            "email": "foo@bar.com already exists in this system.",
            "etc...": "...."
        }
    }
    ```


----------------

### Customer Edit
We'll PATCH to the following endpoint with the following expectations. Unsure at this point whether PATCH or PUT for updating objects will be more appropriate. Assuming PATCH and full data to be sent.

**URL Pattern:** `guidebook-prod.netsuite.com/customer/<ns_customer_id>/`

**Method:** `PATCH`

**Description:** Passing an existing NetSuite customer id, this will allow Guidebook to update existing information about a `Customer`.

**PATCH Request Data:**
```json
{
    "first_name": "Updated First Name",
    "last_name": "Updated Last Name",
    "email": "foo@bar.com",  # can this be updated?
    "guidebook_account_pk": 23423,
    ...,
}
```


**Expected Responses:**
* `200 - OK`:
  * Description: An existing Customer was successfully updated in NS.
  * Response:
    ```json
    {
        "customer_id": 12345
    }
    ```
* `400 - Bad Request`:
  * Description: NS could not update the Customer record b/c invalid or incomplete data was submitted.
  * Response:
    ```json
    {
        "errors": {
            "etc...": "...."
        }
    }
    ```
* `404 - Not Found`:
  * Description: NS could not find the specified `Customer` record.
  * Response:
  ```json
  {}
  ```

------------

### Customer Delete
I'm skeptical that we need this functionality. We currently do not delete `Accounts` in Guidebook (we deactivate them).