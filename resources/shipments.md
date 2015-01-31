# Shipments

Shipments are at the heart of this API. A shipment is an object that represents
a parcel going from A to B.

In order to purchase a label and initiate the physical process of sending a
parcel, there are a few steps to follow:

1. You will have to create a shipment object on our database
2. The API returns a list of rates for this shipment based on the information
   provided
3. You will then have to book the shipment with one of those rates
4. The API will then schedule a collection on the selected date and return the
   documentation to print (labels and customs forms if needed) together with a
   consignment number that can be used to track the progress of the shipment.

You can follow the status of the parcel by using our tracking endpoint and also
get the shipment information from our show endpoint, which will get state
updates.

## Object Definition

These are the fields available on a shipment object.

Attribute | Type | Description
-----|------|--------------
`state`|`string` | The state of the shipment. Valid states documented [here](#valid-states)
`customer_reference`|`string` | A reference to your customer, free form.
`contents`|`string`| The contents of the parcel.
`estimated_value`|`decimal`| The estimated value of the parcel, in GBP, rounded to 2 decimal places. Example: 199.99.
`pickup_date`|`date`| Validation rules for the pickup date can be found [here](#pickup-date)
`label`|`string`| When available, it will be a link to a PDF file.
`customs`|`string`| When available, it will be a link to a PDF file. More information on customs [below](#customs).
`pickup_confirmation`|`string`| Confirmation number for the pickup.
`consignment`|`string`| The consignment number that we will use for tracking.
`liability_amount`|`integer`| The extended liability amount selected. More information on the [liabilities' document](./liabilities.md).
`parcel`|`parcel`| The parcel object. Please find the definition [below](#parcel)
`to_address`|`address`| The destination address of the parcel. Please find the definition [below](#address)
`from_address`|`address`| The collection address of the parcel. Please find the definition [below](#address)

### Valid States

State | Description
-----|--------------
incomplete | The shipment has been created, but not yet booked or acted upon.
completed | The shipment has been booked and a label purchased. Can now be tracked.
cancelled | The shipment has been cancelled before a collection and we have refunded the account.
returned | The shipment was successfully delivered, but was returned as per your customer's request. A return label was purchased.

### Pickup Date

The pickup date's format is YYYY-MM-DD. Please note that weekends and bank
holidays are not allowed.

Pickup dates also depend on cutoff times imposed by carriers. Every rate object
returned by the API will include a cutoff date and time. The cutoff date and time
is the latest a parcel can be collected on the same day.

As an example, let's assume it's 10am. If the cutoff time returned by the API is
today at 11am, then the pickup date can still be today. However, if the cutoff
time returned by the API is 9am, then the earliest pickup date will be tomorrow.

### Customs

Shipments outside the European Union will require a customs form. This is a
declaration of the goods being sent and the reason for it.

One has to print 3 copies of the customs form and:
- give one copy to the courier on pickup
- include the other 2 copies on the parcel

Please note that customs forms are generated asynchronously. This means that you
won't get a link to the customs form immediately on booking an international
shipment.

You will have to fetch the shipment information later on in order to get that
information. It takes about 1 minute to generate it.

### Parcel

The parcel object includes details on the parcel dimensions.

Attribute | Type | Description
-----|------|--------------
`length`|`decimal`| The length of the parcel in centimeters. Example: 10.46.
`width`|`decimal`| The width of the parcel in centimeters. Example: 31.89.
`height`|`decimal`| The height of the parcel in centimeters. Example: 40.00.
`weight`|`decimal`| The weight of the parcel in kilograms. Example: 7.98.

### Address

Attribute | Type | Description
-----|------|--------------
`name`|`string` | The name of the address, for your reference.
`company`|`string` | The name of the company/person at the address.
`phone`|`string` | The mobile phone number for the address. Will be used for SMS notifications regarding tracking.
`line1`|`string` | The first line of the address.
`line2`|`string` | The second line of the address.
`town`|`string` | The name of the town.
`postcode`|`string` | The postcode of the address.
`country_code`|`string` | The country code of the address. More information regarding country codes [here](./countries.md)

## POST /shipments

This endpoint allows you to create a shipment object. This is the first step of
the booking process.

Creating a shipment successfully returns a list of possible rates to choose
from.

### Parameters

Here are the expected parameters:

Attribute | Required | Default
-----|------|--------------
`shipment.customer_reference`| **yes** |
`shipment.contents`| **yes** |
`shipment.estimated_value`| **yes** |
`shipment.pickup_date`| **yes** |
`shipment.liability_amount`| no | Defaults to the free tier value given by the selected carrier.
`shipment.parcel.length`| **yes** |
`shipment.parcel.width`| **yes** |
`shipment.parcel.height`| **yes** |
`shipment.parcel.weight`| **yes** |
`shipment.to_address.name`| **yes** |
`shipment.to_address.company`| no | Empty string.
`shipment.to_address.phone`| **yes** |
`shipment.to_address.line1`| **yes** |
`shipment.to_address.line2`| no | Empty string.
`shipment.to_address.town`| **yes** |
`shipment.to_address.postcode`| **yes** |
`shipment.to_address.country_code`| **yes** |
`shipment.from_address.name`| **yes** |
`shipment.from_address.company`| no | Empty string.
`shipment.from_address.phone`| **yes** |
`shipment.from_address.line1`| **yes** |
`shipment.from_address.line2`| no | Empty string.
`shipment.from_address.town`| **yes** |
`shipment.from_address.postcode`| **yes** |
`shipment.from_address.country_code`| **yes** |
`shipment.customs_form.country_of_manufacture`| **conditional** |
`shipment.customs_form.reason`| **conditional** |
`shipment.customs_form.contents`| **conditional** |

The allowed reasons for customs forms are:

- SALE
- GIFT
- PERSONAL
- DOCUMENTS
- SAMPLE
- REPAIR
- REPLACEMENT
- STOCK

Contents is an array where each element has the following attributes:

Attribute | Required | Default
-----|------|--------------
`description`| **conditional** |
`quantity`| **conditional** |
`value` | **conditional** |

As you can see, the parameters are nested. You can find examples below.
Conditional customs form fields are mandatory on international shipments outside
the EU.

### Examples

A successful request:

    $ curl -i -X POST \
    -H 'Authorization: Token token="<YOUR_TOKEN>"' \
    -H 'Content-Type: application/json' \
    -H 'Accept:application/vnd.parcelbright.v1+json' \
    -d '{
          "shipment":{
            "customer_reference":"123455667",
            "estimated_value":100,
            "contents":"books",
            "pickup_date":"2015-01-29",
            "parcel": {
              "length":10,
              "height":10,
              "width":10,
              "weight":1
            },
            "from_address":{
              "name":"office",
              "postcode":"NW1 0DU",
              "town":"London",
              "phone":"07800000000",
              "line1":"19 Mandela Street",
              "country_code":"GB"
            },
            "to_address": {
              "name":"John Doe",
              "postcode":"E2 8RS",
              "town":"London",
              "phone":"07411111111",
              "line1":"7 Gloucester Square",
              "country_code":"GB"
            }
          }
        }'
    https://api.sandbox.parcelbright.com/shipments

    HTTP/1.1 201 Created
    Connection: close
    Date: Thu, 29 Jan 2015 11:26:06 GMT
    Status: 201 Created
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Location: /shipments/prbf6dfa2b1
    Content-Type: application/json; charset=utf-8
    Etag: W/"ac40b267255e7227b8f1904ec627d1f9"
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: 7cdbb710-eba6-4f5e-a795-9175746bbf7a
    X-Runtime: 6.306231
    Via: 1.1 vegur

    {"shipment":{"state":"incomplete","customer_reference":"123455667","contents":"books","estimated_value":"100.0","pickup_date":"2015-01-29","parcel":{"length":"10.0","width":"10.0","height":"10.0","weight":"1.0"},"from_address":{"name":"office","company":null,"phone":"07800000000","line1":"19 Mandela Street","line2":null,"town":"London","postcode":"NW1 0DU","country_code":"GB"},"to_address":{"name":"John Doe","company":null,"phone":"07411111111","line1":"7 Gloucester Square","line2":null,"town":"London","postcode":"E2 8RS","country_code":"GB"},"rates":[{"code":"N","carrier":"DHL","name":"DOMESTIC EXPRESS","price":"6.29","vat":"1.26","service_type":"collection","transit_days":"1","pickup_date":"2015-01-29","delivery_estimate":"2015-01-30T23:59:00+00:00","cutoff":"2015-01-29T14:00:00+00:00"},{"code":"1","carrier":"DHL","name":"DOMESTIC EXPRESS 12:00","price":"10.61","vat":"2.13","service_type":"collection","transit_days":"1","pickup_date":"2015-01-29","delivery_estimate":"2015-01-30T12:00:00+00:00","cutoff":"2015-01-29T14:00:00+00:00"}],"label":null,"customs":null,"pickup_confirmation":null,"consignment":null,"liability_amount":null}}

Creating an international shipment:

    $ curl -i -X POST \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      -d '{
            "shipment":{
              "parcel": {
                "length":10,
                "height":10,
                "width":10,
                "weight":1
              },
              "customer_reference":"123455667",
              "estimated_value":100,
              "contents":"books",
              "pickup_date":"2015-01-30",
              "from_address":{
                "name":"office",
                "postcode":"NW1 0DU",
                "town":"London",
                "phone":"07800000000",
                "line1":"19 Mandela Street",
                "country_code":"GB"
              },
              "to_address": {
                "name":"John Doe",
                "postcode":"94117-1049",
                "town":"San Francisco",
                "phone":"+1 999999999",
                "line1":"2130 Fulton Street",
                "country_code":"US"
              },
              "customs": {
                "country_of_manufacture":"GB",
                "reason":"SALE",
                "contents":[{
                  "description":"books",
                  "quantity":"1",
                  "value":"100.0"
                }]
              }
            }
          }'
      https://api.sandbox.parcelbright.com/shipments

      HTTP/1.1 201 Created
      Connection: close
      Date: Fri, 30 Jan 2015 15:04:51 GMT
      Status: 201 Created
      X-Frame-Options: SAMEORIGIN
      X-Xss-Protection: 1; mode=block
      X-Content-Type-Options: nosniff
      X-Parcelbright-Media-Type: parcelbright.v1
      Location: /shipments/prba5c3b5aa
      Content-Type: application/json; charset=utf-8
      Etag: W/"eda7ada2a46972b8a4a6de088f861ec6"
      Cache-Control: max-age=0, private, must-revalidate
      X-Request-Id: 485db148-1898-4b5a-a146-f85bb5ee1b88
      X-Runtime: 2.871230
      Via: 1.1 vegur

      {"shipment":{"state":"incomplete","customer_reference":"123455667","contents":"books","estimated_value":"100.0","pickup_date":"2015-01-30","parcel":{"length":"10.0","width":"10.0","height":"10.0","weight":"1.0"},"from_address":{"name":"office","company":null,"phone":"07800000000","line1":"19 Mandela Street","line2":null,"town":"London","postcode":"NW1 0DU","country_code":"GB"},"to_address":{"name":"John Doe","company":null,"phone":"+1 999999999","line1":"2130 Fulton Street","line2":null,"town":"San Francisco","postcode":"94117-1049","country_code":"US"},"rates":[{"code":"D","carrier":"DHL","name":"EXPRESS WORLDWIDE","price":"13.30","vat":"0.00","service_type":"collection","transit_days":"3","pickup_date":"2015-01-30","delivery_estimate":"2015-02-02T23:59:00+00:00","cutoff":"2015-01-30T14:00:00+00:00"},{"code":"T","carrier":"DHL","name":"EXPRESS 12:00","price":"14.62","vat":"0.00","service_type":"collection","transit_days":"3","pickup_date":"2015-01-30","delivery_estimate":"2015-02-02T12:00:00+00:00","cutoff":"2015-01-30T14:00:00+00:00"},{"code":"L","carrier":"DHL","name":"EXPRESS 10:30","price":"19.64","vat":"0.00","service_type":"collection","transit_days":"3","pickup_date":"2015-01-30","delivery_estimate":"2015-02-02T10:30:00+00:00","cutoff":"2015-01-30T14:00:00+00:00"}],"service":null,"label":null,"customs":null,"pickup_confirmation":null,"consignment":null,"liability_amount":null}}

### Response object

Besides the shipment attributes mentioned [above](#object-definition), an
incomplete shipment also includes an array of rates. Each rate has the following
details:

Attribute | Type | Description
-----|------|--------------
`code`|`string`| The code for the rate, which will be use to make a booking.
`carrier`|`string`| The name of the carrier.
`name`|`string`| The name of the service.
`price`|`decimal`| The price of the service. Example: 4.99.
`vat`|`decimal`| The VAT of the service. Example: 1.00.
`service_type`|`string`| The type of service: `collection` or `dropoff`.
`transit_days`|`integer`| The number of transit days for the shipment to arrive.
`pickup_date`|`date`| The actual pickup date for the shipment.
`delivery_estimate`|`datetime`| An estimated time of delivery.
`cutoff`|`datetime`| The cutoff time for this rate, more information [above](#pickup-date).

### Response codes

The statuses returned by this endpoint are:

- `201` - the shipment was successfully created.
- `400` - the information provided is wrong, check the response body for errors.

### Response headers

When a shipment is successfully created, the API will return a `Location` header
with the location of the newly created shipment.

The successful example above includes the following header in the response:

    Location: /shipments/prbf6dfa2b1

This means one will be able to see the shipment attributes if one visits that
path.

## GET /shipments/:slug

This endoint allows you to create a shipment object. This is the first step of
the booking process.

Fetching a valid but incomplete shipment returns a list of possible rates to
choose from.

### Parameters

This endpoint does not have any parameters.

### Response codes

The statuses returned by this endpoint are:

- `200` - the shipment was found and the details are in the body.
- `304` - returned when a conditional get is submitted - it indicates that the
  object didn't change and therefore no details are in the body.
- `404` - we could not find the shipment with the given identifier.

### Response headers

In order to perform conditional GET requests, you will be interested in the
following 2 headers returned:

- `Etag: "5a526eb4bb0d7348563353bb0cd5c356"`
- `Last-Modified: Thu, 29 Jan 2015 11:26:05 GMT`

You can find more information on conditional GET requests [here](../Overview.md).

### Examples

Successful request:

    $ curl -X GET -i \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/shipments/prbf6dfa2b1

    HTTP/1.1 200 OK
    Connection: close
    Date: Thu, 29 Jan 2015 14:13:21 GMT
    Status: 200 OK
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Etag: "5a526eb4bb0d7348563353bb0cd5c356"
    Last-Modified: Thu, 29 Jan 2015 11:26:05 GMT
    Content-Type: application/json; charset=utf-8
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: cfd8f530-21a8-4d52-8fdf-16777143e9ba
    X-Runtime: 1.266285
    Via: 1.1 vegur

    {"shipment":{"state":"incomplete","customer_reference":"123455667", ... }}

We have omitted the rest of the shipment information being returned.

Unknown shipment identifier:

    $ curl -i -X GET \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/shipments/unknown

    HTTP/1.1 404 Not Found
    Connection: close
    Date: Thu, 29 Jan 2015 14:18:57 GMT
    Status: 404 Not Found
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Cache-Control: no-cache
    X-Request-Id: de335c0b-eaad-4c2b-ae08-5f92c418b706
    X-Runtime: 0.499848
    Via: 1.1 vegur

    {"message":"Record not found"}

## POST /shipments/:slug/book

This endpoint allows one to book a shipment that was previously created on the
system with a given rate code.

The result will be a shipment with a label, consignment number, pickup
confirmation code and customs form if needed.

Depending on which carrier one chooses, the pickup confirmation code might not
be available straight away, as there is an asynchronous job running to perform
the booking.

Similarly, if a shipment needs a customs form, this will be generated
asynchronously and therefore you will have to get the shipment information a few
seconds later to see the link to the customs form.

This information will be communicated via the HTTP status codes returned by the
API. More information can be found below.

### Parameters

Name | Type | Required | Description
-----|------|-------------
`rate_code`|`string` | **yes** | The code of the rate from the list returned by the create shipment action.
`liability_amount`|`integer`| no | If you want extended liability on the shipment, this is where you set the prefered amount based on the liabilities endpoint.

### Response codes

The statuses returned by this endpoint are:

- `201` - The shipment was booked successfully and no further action is needed.
  The body of the request includes all the information needed to proceed.
- `202` - The shipment was booked successfully, however we don't yet have all
  the metadata needed by you. This means either the pickup is still being
  confirmed or the customs form is still being generated. We advise to submit a
  GET request a few seconds later to GET the shipment information again until
  those fields are present.
- `400` - There was a validation error, meaning the parameters submitted could
  not be recognised or were missing. Alternatively, you may have insufficient
  funda available. Please check the body of the response for
  further details on the error.
- `404` - We could not find the shipment with the given identifier.

### Response headers

There are no relevant response headers to mention for this action.

### Examples

Successful request:

    $ curl -i -X POST \
      -H 'Authorization: Token token="<YOUR_TOKEN>"'
      -H 'Content-Type: application/json'
      -H 'Accept:application/vnd.parcelbright.v1+json'
      -d '{ "rate_code":"N" }' \
      https://api.sandbox.parcelbright.com/shipments/prb6c8c09d9/book

    HTTP/1.1 202 Accepted
    Connection: close
    Date: Thu, 29 Jan 2015 16:16:09 GMT
    Status: 202 Accepted
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Location: /shipments/prb6c8c09d9
    Content-Type: application/json; charset=utf-8
    Cache-Control: no-cache
    X-Request-Id: 34c85ecc-d243-43f4-bc19-05779fb0f873
    X-Runtime: 7.280311
    Via: 1.1 vegur

    {
      "shipment":{
        "state":"completed",
        "customer_reference":"123455667",
        "contents":"books",
        "estimated_value":"100.0",
        "pickup_date":"2015-01-30",
        "parcel":{
          "length":"10.0",
          "width":"10.0",
          "height":"10.0",
          "weight":"1.0"
        },
        "from_address":{
          "name":"office",
          "company":null,
          "phone":"07800000000",
          "line1":"19 Mandela Street",
          "line2":null,
          "town":"London",
          "postcode":"NW1 0DU",
          "country_code":"GB"
        },
        "to_address":{
          "name":"John Doe",
          "company":null,
          "phone":"07411111111",
          "line1":"7 Gloucester Square",
          "line2":null,
          "town":"London",
          "postcode":"E2 8RS",
          "country_code":"GB"
        },
        "service":{
          "code":"N",
          "carrier":"DHL",
          "name":"DOMESTIC EXPRESS",
          "price":"6.29",
          "vat":"1.26",
          "service_type":"collection"
        },
        "label":"https://pb-labels-sandbox.s3-eu-west-1.amazonaws.com/cofundit-limited/2015/1/29/prb6c8c09d9.pdf",
        "customs":null,
        "pickup_confirmation":null,
        "consignment":"2482606136",
        "liability_amount":50
      }
    }

As you can see from the status code `202`, the shipment is pending a pickup
confirmation which will happen in the next few seconds.

Booking an international shipment:

    $ curl -i -X POST \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      -d '{ "rate_code":"D" }' \
      "https://api.sandbox.parcelbright.com/shipments/prba5c3b5aa/book"

    HTTP/1.1 202 Accepted
    Connection: close
    Date: Fri, 30 Jan 2015 15:19:51 GMT
    Status: 202 Accepted
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Location: /shipments/prba5c3b5aa
    Content-Type: application/json; charset=utf-8
    Cache-Control: no-cache
    X-Request-Id: 5bcf0650-fb35-49b2-b5a9-cb1914b220ad
    X-Runtime: 13.886867
    Via: 1.1 vegur

    {"shipment":{"state":"completed","customer_reference":"123455667","contents":"books","estimated_value":"100.0","pickup_date":"2015-02-03","parcel":{"length":"10.0","width":"10.0","height":"10.0","weight":"1.0"},"from_address":{"name":"office","company":null,"phone":"07800000000","line1":"19 Mandela Street","line2":null,"town":"London","postcode":"NW1 0DU","country_code":"GB"},"to_address":{"name":"John Doe","company":null,"phone":"+1 999999999","line1":"2130 Fulton Street","line2":null,"town":"San Francisco","postcode":"94117-1049","country_code":"US"},"service":{"code":"D","carrier":"DHL","name":"EXPRESS WORLDWIDE","price":"13.30","vat":"0.00","service_type":"collection"},"label":"https://pb-labels-sandbox.s3-eu-west-1.amazonaws.com/cofundit-limited/2015/1/30/prba5c3b5aa.pdf","customs":null,"pickup_confirmation":null,"consignment":"2482609835","liability_amount":50}}

The status code `202` means the customs document is still being generated. Try
to fetch the shipment in a few seconds to get it.

Validation error for missing rate code:

    $ curl -i -X POST \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/shipments/prb3d099690/book

    HTTP/1.1 400 Bad Request
    Connection: close
    Date: Thu, 29 Jan 2015 15:06:43 GMT
    Status: 400 Bad Request
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Cache-Control: no-cache
    X-Request-Id: 636dd788-655e-46c5-9a5e-695a34783d65
    X-Runtime: 0.060501
    Via: 1.1 vegur

    {
      "message":"Validation error",
      "errors":[{
        "resource":"shipment",
        "field":"rate_code",
        "message":":rate_code parameter is missing",
        "code":"parameters.rate_code_missing"
      }]
    }

## GET /shipments/:slug/track

### Parameters

There are no parameters for this endpoint.

### Response codes

The possible status codes are:

- `200` - the shipment was found and the available tracking information is in
  the response body.
- `404` - we could not find the shipment with the given identifier.

### Response headers

There are no relevant response headers to mention for this action.

### Examples

Successful tracking information:

    $ curl -i -X GET \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/shipments/prb6c8c09d9/track

    HTTP/1.1 200 OK
    Connection: close
    Date: Thu, 29 Jan 2015 17:03:25 GMT
    Status: 200 OK
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Etag: W/"1fc5bda6713287cb11b514b6a9836ce8"
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: 6f44c429-1a20-4c76-b91f-00b7e419ccd2
    X-Runtime: 2.495691
    Via: 1.1 vegur

    {
      "events":[{
        "timestamp":"2014-11-27T12:44:00+00:00",
        "description":"Shipment picked up",
        "detail":null,
        "location":"GUADALAJARA - MEXICO"
      },{
        "timestamp":"2014-11-27T13:42:45+00:00",
        "description":"Processed at GUADALAJARA - MEXICO",
        "detail":null,
        "location":"GUADALAJARA - MEXICO"
      }]
    }

Please note that the array of events will be returned in ascending order by the
timestamp field on each event.

### Response object

The tracking response includes an array of tracking events. Each event has the
following details:

Attribute | Type | Description
-----|------|--------------
`timestamp`|`datetime`| Date and time when the event was recorded by the carrier
`description`|`string`| Human-readable description of the event
`detail`|`string`| Any extra details about the event provided by the carrier, or `null` if none are available
`location`|`string`| Location of the tracking event, or `null` if not relevant to this event
