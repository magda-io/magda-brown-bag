# Play with Registry

You can find Registry API docs from:

http://20.43.105.161/api/v0/apidocs/index.html

> You need to replace `20.43.105.161` with the external access domain of your Magda instance

## Get Access

To retrieve data from registry, by default, you don't need to login. However, to create or update records, you do need an admin account and the API key that allows you to make requests on behalf of him. 

You can follow the instruction in [Customise Your Magda Deployment](./customise-deployment.md) to create an Admin user and an API key.

## Create an dummy record

```bash
curl --location --request POST 'http://20.43.105.161/api/v0/registry/records' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8=' \
--data-raw '{
  "id": "my-record-001",
  "name": "test",
  "aspects": {}
}'
```

> You need to replace the `ip`, `X-Magda-API-Key-Id` & `X-Magda-API-Key` with your own value.

> Magda doesn't enforce id format as long as it doesn't collide with any existing records. It's common that people use "[type string]-[uuid]" format. 

> We didn't supply any "aspect" data yet as we haven't define the aspect (and its schema) yet (API will report an error if you do so). A record by itself doesn't carry any useful info  without "aspects". We can't even tell if this record is an order unless it carries [order-details aspect](./order-details-schema.json) that we are going to define.

## Define the `order-details` Aspect

Before you can attach "order-details" aspect data to any record (and make it an "order" record), you need to define / register the aspect (with its JSON schema) with registry API.

You can find the sample JSON schema [here](./order-details-schema.json)


```bash
curl --location --request POST 'http://20.43.105.161/api/v0/registry/aspects' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8=' \
--data-raw '{
  "id": "order-details",
  "name": "test",
  "jsonSchema": {
    "$schema": "http://json-schema.org/schema#",
    "title": "Product Order Details",
    "description": "Provides information of the produce order.",
    "type": "object",
    "properties": {
      "productName": {
        "title": "Name of the product",
        "type": "string"
      },
      "quantity": {
        "title": "The quantity of this order",
        "minimum": 1,
        "type": "number"
      },
      "unitPrice": {
        "title": "Unit Price of the product",
        "minimum": 1,
        "type": "number"
      },
      "notes": {
        "title": "the optional notes of the order",
        "type": "string"
      }
    },
    "required": ["productName", "quantity", "unitPrice"]
  }
}
'
```

> You need to replace the `ip`, `X-Magda-API-Key-Id` & `X-Magda-API-Key` with your own value.

## Update Record with Aspect 

We updated the record name and attach the order-details aspects.

```bash
curl --location --request PUT 'http://20.43.105.161/api/v0/registry/records/my-record-001' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8=' \
--data-raw '{
  "id": "my-record-001",
  "name": "test order",
  "aspects": {
      "order-details": {
          "productName": "apple",
          "quantity": 2,
          "unitPrice": 1.5
      }
  }
}'
```

> You need to replace the `ip`, `X-Magda-API-Key-Id` & `X-Magda-API-Key` with your own value.


The customer may change his mind and we might need to update the order:

```bash
curl --location --request PUT 'http://20.43.105.161/api/v0/registry/records/my-record-001' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8=' \
--data-raw '{
  "id": "my-record-001",
  "name": "test order",
  "aspects": {
      "order-details": {
          "productName": "pear",
          "quantity": 3,
          "unitPrice": 2.2,
          "notes": "Customer changed his mind"
      }
  }
}'
```

## Retrieve Record History

Magda is built with [event sourcing model](https://martinfowler.com/eaaDev/EventSourcing.html). i.e. Any data changes will trigger events which form an event stream which present the state of the system.

You can retrieve event history of a record and bring back its state at a point in time from the history API.

```bash
curl --location --request GET 'http://20.43.105.161/api/v0/registry/records/my-record-001/history' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8=' \
```

> This request will list events of the records in [JSON patch](http://jsonpatch.com/) format


> And the request below will bring back the previous "Apple" order back:

```bash
curl --location --request GET 'http://20.43.105.161/api/v0/registry/records/my-record-001/history/105199' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8='
```