# Liabilities

## GET /liabilities/:carrier

This request returns the extended liability options for the selected carrier.

Every shipment includes default liability when nothing else is selected. This
means that if a parcel gets lost or is damaged, we are able to submit a claim
and get some money back. That amount depends on the carrier selected, but it
typically ranges between £50 and £100.

If you wish to insure the parcel for more than that default cover, you can buy
extended liability. This endpoint provides the several options available for the
selected carrier and how much each option will cost.

When you book a shipment, you can then submit information regarding which
extended liability to use.

### Parameters

Name | Type | Description
-----|------|-------------
`carrier`|`string` | The name of the carrier

The allowed carriers are:

- dhl
- ukmail
- ups
- dpd
- interlink
- collectplus

### Examples

Successful request:

    $ curl -i -X GET \
      -H 'Authorization: Token token="18a958318f0c26f63e0e79122d378f19"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/liabilities/dhl

    HTTP/1.1 200 OK
    Connection: close
    Date: Wed, 28 Jan 2015 17:35:25 GMT
    Status: 200 OK
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Etag: W/"e99f1058cb64ec58392762b9c1054796"
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: 39e8005f-41ec-4da5-819c-cd1241d48d61
    X-Runtime: 0.028056
    Via: 1.1 vegur

    {"liabilities":[{"cost":"0.00","amount":"50.00","currency":"GBP"},{"cost":"7.50","amount":"500.00","currency":"GBP"},{"cost":"13.00","amount":"1000.00","currency":"GBP"}]}

Unknown carrier:

    $ curl -i -X GET \
      -H 'Authorization: Token token="18a958318f0c26f63e0e79122d378f19"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/liabilities/unknown

    HTTP/1.1 404 Not Found
    Connection: close
    Date: Wed, 28 Jan 2015 17:47:00 GMT
    Status: 404 Not Found
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Cache-Control: no-cache
    X-Request-Id: 46ee122e-b6a0-4202-9552-2c7be7ce6f68
    X-Runtime: 0.063997
    Via: 1.1 vegur

    {"message":"Carrier not found"}
