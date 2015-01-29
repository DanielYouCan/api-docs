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

This endoint allows you to create a shipment object. This is the first step of
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

As you can see, the parameters are nested. You can find examples below.

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

Error responses will return:

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

TODO

## POST /shipments/:slug/book

TODO

## GET /shipments/:slug/track

TODO
