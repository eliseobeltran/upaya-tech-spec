# Product API
We'll need an endpoint to create and update `Products` in our product catalog in order to sync them with the product catalog in Guidebook.

We imagine the catalog to be fairly static, in that new products are added infrequently and updates to existing products are quite rare as well.

[We have decided that *we will never delete* `Products` from our catalog](https://confluence.guidebook.com/display/WT/Open+Billing+Questions) on our side. If we ever need to deprecate a `Product` it will be marked as inactive and we'll have internal tooling to migrate `Items` associated to those `Products`.


### Open Questions
**Does NS Care About Our Product Catalog:**
Does NetSuite even care about our product catalog? Is it required to create `Invoices`?

**Do We Need Endpoints For This:**
Since our product catalog will remain fairly static, do we even need endpoints to keep the two catalogs (in Guidebook and in NS) in sync? Can we simply manage the product catalog in NetSuite via their web interface?

------------

### Product Creation
This endpoint will allow us to create new `Product` objects in NetSuite. Ideally, upon successful `Product` creation, NS will return the `pk` of the Product in NS so Guidebook can persist it on our end for future calls.

**URL Pattern:** `guidebook-prod.netsuite.com/product/`

**Method:** `POST`

**POST Request Data:**
```json
{
    "name": "Plus Guide",
    "price": {
        "US:USD": 300,
        "KR:KRW": 450,
        "GB:GBP": 500
    }
}
```

**Expected Responses:**
  * `201 - Created`:
    * Description: A new Product was successfully created in NS.
    * Response:
    ```json
    {
        "product_id": 12345
    }
  ```
  * `400 - Bad Request`:
    * Description: NS could not create a new Product record b/c invalid or incomplete data was submitted.
    * Response:
    ```json
    {
        "errors": {
            "name": "A product with this name already exists in the system.",
        }
    }
    ```
