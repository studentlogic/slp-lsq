# StudentLogic and LeadSquared Integration

A documentation for the integration with LeadSquared and description of the API provided by StudentLogic for pushing the prospect details.

### OAuth2 Authentication
There are 2 options available - either using account credentials (client_id with client_password) or authentication using a user account. The first option doesn't require end user involvement and it's preferable for the testing/UAT phase.

You will need the client_id and secret. This is available in the StudentLogic system settings (requires root level access) but as this is a 3rd party integration, you can receive it from the support.

##### Receiving Token
To receive the authentication token, call the `/slp-lsq/oauth/access_token` endpoint:

  curl https://uat.studentlogic.pro/slp-lsq/oauth/access_token -X POST -d "client_id=your-client-id" -d "client_secret=your-client-secret" -d "grant_type=client_credentials"
  
You should receive something like this:

    {"token_type":"Bearer",
    "access_token":"8g9TvYKiF0g2uWRuf01VOS719QNdJnwnpkB2nKzv",
    "expires_in":3599,
    "refresh_token":"mQBeTpp2f8eVGbFOM5LYBVmZOrLQCPgy2kVOX2kV"}

Where the `access_token` is to be used in the header `Authorization: Bearer ` and the `refresh_token` is used to refresh the expired token.

##### Refreshing Token
Using the `refresh_token` received above:

    curl https://uat.studentlogic.pro/slp-lsq/oauth/access_token -X POST -d "client_id=your-client-id" -d "client_secret=your-client-secret" -d "refresh_token=${refresh_token}" -d "grant_type=refresh_token"
    
#### Accessing Protected Resources
Using the `access_token` received above:

    curl --dump-header - -H "Authorization: Bearer ${access_token}" https://uat.studentlogic.pro/slp-lsq/verify-token

Or just:
    
    curl -H "Authorization: Bearer ${access_token}" https://uat.studentlogic.pro/slp-lsq/verify-token
    
The first options shows headers and it's useful for debugging. Successful request will return headers similar to:
  
    HTTP/1.1 200 OK
    Vary: Origin
    Referrer-Policy: origin-when-cross-origin, strict-origin-when-cross-origin
    X-Frame-Options: DENY
    X-XSS-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Permitted-Cross-Domain-Policies: master-only
    Date: Wed, 09 Sep 2020 06:46:21 GMT
    Content-Type: application/json
    Content-Length: 85

Using an invalid token will result in:

    HTTP/1.1 401 Unauthorized
    Vary: Origin
    Referrer-Policy: origin-when-cross-origin, strict-origin-when-cross-origin
    X-Frame-Options: DENY
    WWW-Authenticate: Bearer error="invalid_token", error_description="The access token is not found"
    X-XSS-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Permitted-Cross-Domain-Policies: master-only
    Date: Wed, 09 Sep 2020 06:47:52 GMT
    Content-Length: 0

And using expired token will result in:

    HTTP/1.1 401 Unauthorized
    Vary: Origin
    Referrer-Policy: origin-when-cross-origin, strict-origin-when-cross-origin
    X-Frame-Options: DENY
    WWW-Authenticate: Bearer error="invalid_token", error_description="The access token expired"
    X-XSS-Protection: 1; mode=block
    X-Content-Type-Options: nosniff
    X-Permitted-Cross-Domain-Policies: master-only
    Date: Wed, 09 Sep 2020 06:50:38 GMT
    Content-Length: 0

The `error_description` provides the details.


### CRM API (simplified)
This is a simplified API that only allows creating of prospects.

The end point is `/slp-lsq/api/crm/parents`

To create a new contact you need to issue a POST request with the following content:

    POST https://uat.studentlogic.pro/slp-lsq/api/crm/parents
    Content-Type: application/json
    Accept: text/json
    Authorization: Bearer xnAyft44nhAAtxxH56T99shgfFNnAOUTZpgRa95q

    {
      "parent": {
        "lsq_uuid": "646c7c81-ebbd-440a-8894-a0f3ebd106ea",
        "tenant_id": 1,
        "name": "Testy 3",
        "email": "testy@nl-abc.com",
        "primary_relation_id": 2,
        "phone": "6512345001"
      },
      "students": [
        {
          "name": "Testy Kid 3",
          "gender_id": 1,
          "date_of_birth": "2015-01-05"
        }
      ],
      "address": {
        "address1": "Testy Address",
        "address2": "Line 2",
        "address3": "Line 3",
        "city": "Singapore",
        "country_id": 65
      }
    }

For the `parent` object, all the fields are required. `students` accepts an array - again all fields are required. At the moment, the system will accept empty array (but it must be present), although it's preferable to create a complete family record. `address` is optional (can be omitted) and each fields is optional as well.

##### Validation
- phone - must follow format 65xxxxxxxx (65 followed by 8 digits - no spaces or other characters) (this is the required format for the SMS system)
- email - must be a valid email
- both phone and email must be unique
- lsq_uuid - LSQ identification of the record - I believe this is the `RelatedId` 
- tenant_id - must be a valid branch ID - the list of branches can be obtained here:

      curl -H "Authorization: Bearer ${access_token}" https://uat.studentlogic.pro/slp-lsq/api/settings/tenants
      
- primary_relation_id - must be a valid relationship type ID - the list can be obtained here:

      curl -H "Authorization: Bearer ${access_token}" https://uat.studentlogic.pro/slp-lsq/api/settings/relation_types

- country_id - must be either null or a valid country ID - the list can be obtained here:

      curl -H "Authorization: Bearer ${access_token}" https://uat.studentlogic.pro/slp-lsq/api/settings/countries
- gender_id - 1 for male or 2 for female
