# Users

To begin using the API to create shipments etc, a user will need to be created. Once created, a user will have an API token 
associated with themself and this then can be used to make subsequent API calls. 

1. Create user, providing user details such as collection address and company info

## Endpoints

- Create a user: [POST /users](#post-users)

### Parameters

Here are the expected parameters:

Attribute | Required | Default
-----|------|--------------
`user.name`| **yes** |
`user.email`| **yes** |
`user.password`| **yes** |
`company.name`| **yes** |
`company.currency_code` | **yes** |
`shipment.addresses[n].value`| **yes** |
`shipment.addresses[n].name`| **no** |
`shipment.addresses[n].company`| **no** |
`shipment.addresses[n].phone`| **yes** |
`shipment.addresses[n].email`| **no** |
`shipment.addresses[n].line1`| **yes** |
`shipment.addresses[n].town`| **yes** |
`shipment.addresses[n].postcode`| **yes** |
`shipment.addresses[n].country_code`| **yes** |

### Examples

A successful request:

    $ curl -i -X POST \
    -H 'Content-Type: application/json' \
    -H 'Accept:application/vnd.parcelbright.v1+json' \
    -d '{
          "user":{
            "name":"Shipper", 
            "email":"ship@me.com", 
            "password": "SECURE_PASSWORD"
            }, 
          "company":{
            "name":"Big Corp", 
            "phone":"07411111111"
          }, 
          "addresses":[
            {
              "name":"Office", 
              "company":"My Corp", 
              "phone":"07411111111", 
              "email":"office@mycorp.com", 
              "line1":"7 My Street", 
              "line2":"Some Court", 
              "town":"London", 
              "postcode":"E1 6BT", 
              "country_code":"GB"
            }
          ]
        }
        '
    https://api.sandbox.parcelbright.com/users

    HTTP/1.1 201 Created
    Connection: close
      Date: - Tue, 08 Mar 2016 16:12:24 GMT
      X-Frame-Options:- SAMEORIGIN
      X-Xss-Protection:- 1; mode=block
      X-Content-Type-Options:- nosniff
      X-Parcelbright-Media-Type:- parcelbright.v1
      Content-Type:- application/json; charset=utf-8
      Vary: - Accept-Encoding
      Content-Encoding:- gzip
      Etag:- W/"f07693a291259dce3100febe6c4fe710"
      Cache-Control:- max-age=0, private, must-revalidate
      X-Request-Id:- 23caef70-45b5-4c11-88dd-bcf0a9823b90
      X-Runtime:- '1.528241'
      Via:- 1.1 vegur

    {
      "token_type":"bearer", 
      "access_token":"fb1ada62131627dee12dad3a3b1c562a", 
      "email":"ship@me.com", 
      "password":"SECURE_PASSWORD"
    }

--

Omitting required parmeters

    $ curl -i -X POST \
    -H 'Content-Type: application/json' \
    -H 'Accept:application/vnd.parcelbright.v1+json' \
    -d '{
          "user":{
            "name":"", 
            "email":"ship@me.com", 
            "password": "SECURE_PASSWORD"
            }, 
          "company":{
            "name":"Big Corp", 
            "phone":"07411111111"
          }, 
          "addresses":[
            {
              "name":"Office", 
              "company":"My Corp", 
              "phone":"07411111111", 
              "email":"office@mycorp.com", 
              "line1":"7 My Street", 
              "line2":"Some Court", 
              "town":"London", 
              "postcode":"E1 6BT", 
              "country_code":"GB"
            }
          ]
        }
        '
    https://api.sandbox.parcelbright.com/users

    status:
      code: 400
      message: Bad Request
    headers:
      Server:
      - Cowboy
      Connection:
      - close
      Date:- Tue, 08 Mar 2016 16:12:25 GMT
      X-Frame-Options:- SAMEORIGIN
      X-Xss-Protection:- 1; mode=block
      X-Content-Type-Options:- nosniff
      X-Parcelbright-Media-Type:- parcelbright.v1
      Content-Type:- application/json; charset=utf-8
      Vary:- Accept-Encoding
      Content-Encoding:- gzip
      Cache-Control:- no-cache
      X-Request-Id:- 38dc7abb-dd0d-4be5-b002-cdb9a867f307
      X-Runtime:- '0.573243'
      Via:- 1.1 vegur

    {"message"=>{"company.name"=>["Required"]}}
