# Countries

## GET /countries

We provide a list of countries, including their name and code, so that you can
use the correct country code when booking a shipment.

Each country also has an `eu` boolean flag, indicating whether or not it is currently part of the European Union. This is to allow API users to determine whether a shipment requires a customs form. Customs forms are only required for shipments to countries which are outside the EU.

Country codes returned via the API comply with the [ISO 3166
alpha2](http://www.iso.org/iso/country_codes.htm) international standard.

### Parameters

There are no parameters on this request.

### Examples

Here is an example of a successful request:

    $ curl -i -X GET \
      -H 'Authorization: Token token="<YOUR TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/countries

    HTTP/1.1 200 OK
    Connection: close
    Date: Thu, 29 Jan 2015 10:30:07 GMT
    Status: 200 OK
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Etag: "cfb60c3ac577e77535287963e1780bbb"
    Content-Type: application/json; charset=utf-8
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: 03b11930-236c-47fc-b34b-a251cadbf89c
    X-Runtime: 2.229218
    Via: 1.1 vegur

    {"countries":[{"name":"United Kingdom","code":"GB","eu":true}, ... ]}

We have ommited the entire list, however the API returns about 250 countries.
