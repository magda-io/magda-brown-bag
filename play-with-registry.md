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


> And the request below will bring the previous "Apple" order back:

```bash
curl --location --request GET 'http://20.43.105.161/api/v0/registry/records/my-record-001/history/105199' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8='
```

## What about datasets?

Magda is a data catelog tool. All dataset related function is built on top of the following built-in aspects:
- [dcat-dataset-strings](https://github.com/magda-io/magda/blob/master/magda-registry-aspects/dcat-dataset-strings.schema.json)
- [dataset-distributions](https://github.com/magda-io/magda/blob/master/magda-registry-aspects/dataset-distributions.schema.json)
- [dcat-distribution-strings](https://github.com/magda-io/magda/blob/master/magda-registry-aspects/dcat-distribution-strings.schema.json)

The schema above is created based on [Data Catalog Vocabulary (DCAT)](https://www.w3.org/TR/vocab-dcat/). You also can find more aspect schema from [here](https://github.com/magda-io/magda/tree/master/magda-registry-aspects), which are used by different components of the system to further enhance the metadata.

### Create a sample distribution

> A dataset may contains one or more distributions and each of the distributions could either be a data file or an API.

> Magda has an internal [storage API](https://github.com/magda-io/magda/tree/master/deploy/helm/internal-charts/storage-api) which is backed by [minio](https://min.io/). However, we focus on data federation and don't force you store the data with us.

> The request below will create a sample dsitribution that we will later link to a dataset. You don't have to create / define aspect before send this request. Those aspect should be created by the connector that is included in our customised deployment.

```bash
curl --location --request POST 'http://20.43.105.161/api/v0/registry/records' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8=' \
--data-raw '{
  "id": "my-distribution-001",
  "name": "test distribution 001",
  "aspects": {
    "dcat-distribution-strings": {
      "description": "Credit Lisence data....",
      "downloadURL": "https://data.gov.au/data/dataset/fa0b0d71-b8b8-4af8-bc59-0b000ce0d5e4/resource/548075a1-c36e-4837-bf26-cc00567b5c23/download/credit_lic_202105.csv-geo-au",
      "format": "CSV-GEO-AU",
      "issued": "2015-12-23T14:35:33Z",
      "license": "Creative Commons Attribution 3.0 Australia",
      "modified": "2021-05-03T01:50:35Z",
      "title": "Test Credit Licence Dataset"
    }
  }
}'
```

### Create a sample dataset

> The following request will create a dataset and link the previous "distribution" to it via "dataset-distributions" aspect.

```bash
curl --location --request POST 'http://20.43.105.161/api/v0/registry/records' \
--header 'Content-Type: application/json' \
--header 'X-Magda-API-Key-Id: 6a74723b-2e73-490d-9cb5-7e5e7af3f7e6' \
--header 'X-Magda-API-Key: GLv3sLlSMUTnpJV+Jqgy18ckOdI4hghxEWkaxd8pzk8=' \
--data-raw '{
  "id": "my-dataset-001",
  "name": "test dataset 001",
  "aspects": {
    "dcat-dataset-strings": {
      "accrualPeriodicity": "monthly",
      "contactPoint": "access.request@example.com",
      "description": "This is a test dataset",
      "issued": "2015-12-21T22:19:54Z",
      "keywords": [
        "Test",
        "Dataset"
      ],
      "landingPage": "https://example.com",
      "languages": ["English"],
      "modified": "2021-05-03T01:50:39Z",
      "publisher": "Test Org",
      "temporal": {
        "end": "",
        "start": "2015-12-21"
      },
      "themes": [],
      "title": "test dataset 001"
    },
    "dataset-distributions": {
        "distributions": ["my-distribution-001"]
    }
  }
}
'
```

> Magda's frontend is a single-page application (SPA). You probably can watch its network requests to learn more.

> We only covered a small set of registry API. More functions (e.g. create webhooks) can be found from the API docs (see link above).