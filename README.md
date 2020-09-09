# StudentLogic and LeadSquare Integration

A documentation for the integration with LeadSquare and description of the API provided by StudentLogic for pushing the prospect details.

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

