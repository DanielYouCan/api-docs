# API Overview

This is an overview of the API structure and pre-conditions.

Please note that all examples provided use our sandbox environment's endpoint
`https://api.sandbox.parcelbright.com`. In a production environment, the only
difference will be this endpoint, which will be `https://api.parcelbright.com`.

# Overview

1. [Versioning](#versioning)
2. [Schema](#schema)
3. [Parameters](#parameters)
4. [Client Errors](#client-errors)
5. [HTTP Redirects](#http-redirects)
6. [HTTP Verbs](#http-verbs)
7. [Authentication](#authentication)
8. [Conditional Requests](#conditional-requests)
9. [Timestamp format](#timestamp-format)

## Versioning

By default, all requests receive the latest version of the API.
We encourage you to explicitly request this version via the `Accept`
header:

    Accept: application/vnd.parcelbright.v1+json

The API responds with a header indicating the served version:

    X-ParcelBright-Media-Type: parcelbright.v1

When we release new versions, the number will increase and the default version
will be the latest one.

## Schema

All API access is over HTTPS, and accessed from the domains mentioned above.
All data is sent and received as JSON. Here is an example of a request.

    $ curl -i -X GET \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.parcelbright.com/liabilities/dhl

    HTTP/1.1 200 OK
    Connection: close
    Date: Wed, 28 Jan 2015 12:34:52 GMT
    Status: 200 OK
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Etag: W/"e99f1058cb64ec58392762b9c1054796"
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: 6fbe73a7-3a80-413b-9c9b-9bb2f5f867de
    X-Runtime: 0.692776
    Via: 1.1 vegur

    {"liabilities":[{"cost":"0.00","amount":"50.00","currency":"GBP"},{"cost":"7.50","amount":"500.00","currency":"GBP"},{"cost":"13.00","amount":"1000.00","currency":"GBP"}]}%

Blank fields are included as `null` instead of being omitted.

## Parameters

Some API methods take optional parameters. For GET requests, any parameters not
specified as a segment in the path can be passed as an HTTP query string
parameter:

    $ curl -i -X POST \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/shipments/prb12345/book?rate_code=N

In this example, `:rate_code` is passed in the query string.

For POST, PATCH, PUT, and DELETE requests, parameters not included in the URL
should be encoded as JSON with a Content-Type of 'application/json':

    $ curl -i -X POST \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      -d '{ "rate_code": "N" }' \
      https://api.sandbox.parcelbright.com/shipments/prb12345/book

## Client Errors

### Invalid JSON sent

Sending invalid JSON will result in a `400 Bad Request` response:

    $ curl -i -X POST \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      -d '{ "invalid": ' \
      https://api.sandbox.parcelbright.com/shipments/prb12345/book

    HTTP/1.1 400 Bad Request
    Connection: close
    Date: Wed, 28 Jan 2015 12:47:01 GMT
    Status: 400 Bad Request
    Content-Type: application/json
    X-Request-Id: 2cf2e8be-bf57-48f8-acc8-176d9057a1ea
    X-Runtime: 0.220175
    Via: 1.1 vegur

    {"message":"Format error","error":"Invalid JSON submitted"}

### Missing parameters

Omitting required parameter values will result in a `400 Bad Request`
response:

    curl -i -X POST \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      -d {} \
      https://api.parcelbright.com/shipments

    HTTP/1.1 400 Bad Request
    Connection: close
    Date: Wed, 28 Jan 2015 12:53:46 GMT
    Status: 400 Bad Request
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Cache-Control: no-cache
    X-Request-Id: 799c1dd0-8bbd-48bc-824b-2c6869280c41
    X-Runtime: 0.201534
    Via: 1.1 vegur

    {"message":"Validation error","errors":[{"resource":"shipment","field":"shipment","message":":shipment parameter missing","code":"parameters.shipment_missing"}]}

Validation errors will include:

Parameter | Type | Description
-----|------|--------------
`message`|`string` | A description of the type of error
`errors`|`array` | An array of errors

An error has the following structure:

Parameter | Type | Description
-----|------|--------------
`resource`|`string` | The name of the resource with an error, e.g. `shipment`
`field`|`string`| The field with the error, e.g. `postcode`
`message`|`string`| A human readable error message
`code`|`string`| The unique error code for the error given

### Unknown endpoint

Requesting an unknown endpoint results in a `404 Not Found` response:

    $ curl -i -X GET \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/unknown

    HTTP/1.1 404 Not Found
    Connection: close
    Date: Wed, 28 Jan 2015 13:01:33 GMT
    Status: 404 Not Found
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Cache-Control: no-cache
    X-Request-Id: b3c16bdd-e198-4d23-96ba-a98449035f4c
    X-Runtime: 0.055741
    Via: 1.1 vegur

    {"message":"Not found"}

## HTTP Redirects

The API uses HTTP redirection where appropriate. Clients should assume that any
request may result in a redirection. Receiving an HTTP redirection is *not* an
error and clients should follow that redirect. Redirect responses will have a
`Location` header field which contains the URI of the resource to which the
client should repeat the requests.

Status Code | Description
-----------|-----------|
`301` | Permanent redirection. The URI you used to make the request has been
superseded by the one specified in the `Location` header field. This and all
future requests to this resource should be directed to the new URI.
`302` | Temporary redirection. The request should be repeated verbatim to the
URI specified in the `Location` header field but clients should continue to use
the original URI for future requests.

Other redirection status codes may be used in accordance with the HTTP 1.1 spec.

## HTTP Verbs

Where possible, the API strives to use appropriate HTTP verbs for each
action.

Verb | Description
-----|-----------
`HEAD` | Can be issued against any resource to get just the HTTP header info.
`GET` | Used for retrieving resources.
`POST` | Used for creating resources.
`PATCH` | Used for updating resources with partial JSON data.
`PUT` | Used for replacing resources.
`DELETE` |Used for deleting resources.

## Authentication

Authentication is achieved using an API token. The token is sent in an HTTP
header like this:

    $ curl -X GET \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      https://api.sandbox.parcelbright.com

### Failed authentication

Authenticating with invalid credentials will return `401 Unauthorized`:

    curl -i -X GET \
      -H 'Authorization: Token token="invalid"' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com

    HTTP/1.1 401 Unauthorized
    Connection: close
    Date: Wed, 28 Jan 2015 13:05:37 GMT
    Status: 401 Unauthorized
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    X-Request-Id: aacfe036-682a-410b-8590-be127ec69cd6
    X-Runtime: 0.846776
    Via: 1.1 vegur

    {"message":"Unauthorized"}

## Conditional requests

Most responses return an `ETag` header. Many responses also return a
`Last-Modified` header. You can use the values
of these headers to make subsequent requests to those resources using the
`If-None-Match` and `If-Modified-Since` headers, respectively. If the resource
has not changed, the server will return a `304 Not Modified`.

Viewing a record:

    curl -i -X GET \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      https://api.sandbox.parcelbright.com/shipments/prb3d099690

    HTTP/1.1 200 OK
    Connection: close
    Date: Wed, 28 Jan 2015 13:22:36 GMT
    Status: 200 OK
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Content-Type: application/json; charset=utf-8
    Etag: W/"a3ce2980ddb29041fcaf9bd26d122a26"
    Last-Modified: Wed, 28 Jan 2015 13:17:53 GMT
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: a934c4e7-108e-4214-9c82-73010829908b
    X-Runtime: 0.157741
    Via: 1.1 vegur

    {"shipment": ... }

Using `If-None-Match`:

    $ curl -i -X GET \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      -H 'If-None-Match: W/"a3ce2980ddb29041fcaf9bd26d122a26"' \
      https://api.sandbox.parcelbright.com/shipments/prb3d099690

    HTTP/1.1 304 Not Modified
    Content-Length: 0
    Connection: keep-alive
    Date: Wed, 28 Jan 2015 13:33:03 GMT
    Status: 304 Not Modified
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Etag: W/"a3ce2980ddb29041fcaf9bd26d122a26"
    Last-Modified: Wed, 28 Jan 2015 13:17:53 GMT
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: 7ec583ae-25d8-47ca-b1e2-a4b8fd54027e
    X-Runtime: 0.160958
    Via: 1.1 vegur

Using `If-Modified-Since`:

    $ curl -i -X GET \
      -H 'Authorization: Token token="<YOUR_TOKEN>"' \
      -H 'Content-Type: application/json' \
      -H 'Accept:application/vnd.parcelbright.v1+json' \
      -H 'If-Modified-Since: Wed, 28 Jan 2015 13:17:53 GMT' \
      https://api.sandbox.parcelbright.com/shipments/prb3d099690

    HTTP/1.1 304 Not Modified
    Content-Length: 0
    Connection: keep-alive
    Date: Wed, 28 Jan 2015 15:48:08 GMT
    Status: 304 Not Modified
    X-Frame-Options: SAMEORIGIN
    X-Xss-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Parcelbright-Media-Type: parcelbright.v1
    Etag: "08913a726b190c9395b8df542d76ec04"
    Last-Modified: Wed, 28 Jan 2015 13:17:53 GMT
    Cache-Control: max-age=0, private, must-revalidate
    X-Request-Id: e08b42f1-0642-41d6-8dd5-314b1172f735
    X-Runtime: 0.369615
    Via: 1.1 vegur

## Timestamp format

Any timestamps returned by the API will conform with the ISO8601 format.

These timestamps look something like `2014-02-27T15:05:06+01:00`.
