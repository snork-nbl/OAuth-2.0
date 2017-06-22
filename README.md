# OAuth 2.0

This library provides OAuth 2.0 authentication and authorization flows. It supports the following flows:

- [OAuth2.JWTProfile.Client](#oauth2jwtprofileclient) &mdash; OAuth 2.0 with the JSON Web Token (JWT) Profile for Client Authentication and Authorization Grants as defined in [IETF RFC 7523](https://tools.ietf.org/html/rfc7523).
- [OAuth2.DeviceFlow.Client](#oauth2deviceflowclient) &mdash; OAuth 2.0 Device Flow for browserless and input-constrained devices. The implementation conforms to the [IETF draft device flow specification](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-05).

The library exposes retrieved access tokens for applications and hides provider-specific operations, including the renewal of expired tokens.

**To add this library to your project, add** `#require "OAuth2.agent.lib.nut:1.0.0"` **to the top of your agent code.**

## OAuth2.JWTProfile.Client

This class implements an OAuth 2.0 client flow using a JSON Web Token (JWT) as the means for requesting access tokens and for client authentication.

**Note** The flow requires an RSA-SHA256 signature which is not currently supported by the Electric Imp [imp API](https://electricimp.com/docs/api/). As a temporary solution we suggest that you use an [AWS Lambda](https://aws.amazon.com/lambda) function that will perform [RSA-SHA256 signatures](examples#amazon-lambda-for-rsa-sha256-signatures) for an agent. However, please note that AWS Lambda is subject to a service charge so you should refer to the Amazon pricing [page](https://aws.amazon.com/lambda/pricing/) for more information before proceeding.

## OAuth2.JWTProfile.Client Usage

### constructor(*providerSettings, userSettings*)

The constructor creates an instance of the *OAuth2.JWTProfile.Client* class. The first parameter, *providerSettings*, is a table that contains provider-specific settings:

| Parameter | Type | Use | Description |
| --- | --- | --- | --- |
| *tokenHost* | String | Required | The token endpoint. This is used by the client to exchange an authorization grant for an access token, typically with client authentication |

The second parameter, *userSettings*, defines a table with user- and application-specific settings:

| Parameter | Type | Use | Description |
| --- | --- | --- | --- |
| *iss* | String | Required | The JWT issuer |
| *scope* | String | Required | A scope. Scopes enable your application to request access only to the resources that it needs while also enabling users to control the amount of access that they grant to your application |
| *jwtSignKey* | String | Required | A JWT sign secret key |
| *rs256signer* | *[AWSLambda](https://github.com/electricimp/awslambda)* | Required | Instance of [AWSLambda](https://github.com/electricimp/awslambda) for RSA-SHA256 encryption. You can use [this example code](examples#jwt-profile-for-oauth-20) to create the AWS Lambda function |
| *sub* | String | Optional. *Default:* the value of *iss* | The subject of the JWT. **Note** Google seems to ignore this field |

**Note** When omitted, the optional *sub* property is substituted by the mandatory *iss* property.

#### JWT Profile Client Creation Example

```squirrel
// AWS Lambda libraries
#require "AWSRequestV4.class.nut:1.0.2"
#require "AWSLambda.agent.lib.nut:1.0.0"

// OAuth 2.0 library
#require "OAuth2.agent.lib.nut:1.0.0"

// Substitute with real values
const LAMBDA_REGION        = "us-west-1";
const LAMBDA_ACCESS_KEY_ID = "<AWS access key id>";
const LAMBDA_ACCESS_KEY    = "<AWS access key>";
const GOOGLE_ISS           = "rsalambda@quick-cacao-168121.iam.gserviceaccount.com";
const GOOGLE_SECRET_KEY    = "-----BEGIN PRIVATE KEY-----\nprivate key goes here\n-----END PRIVATE KEY-----\n";

// Create AWS Lambda Instance
local signer = AWSLambda(LAMBDA_REGION, LAMBDA_ACCESS_KEY_ID, LAMBDA_ACCESS_KEY);

local providerSettings =  {
    "tokenHost"  : "https://www.googleapis.com/oauth2/v4/token"
};

local userSettings = {
    "iss"         : GOOGLE_ISS,
    "jwtSignKey"  : GOOGLE_SECRET_KEY,
    "scope"       : "https://www.googleapis.com/auth/pubsub",
    "rs256signer" : signer
};

local client = OAuth2.JWTProfile.Client(providerSettings, userSettings);
```

**Important** The name of the AWS Lambda function **must** be `RSALambda`.

## OAuth2.JWTProfile.Client Methods

### acquireAccessToken(*tokenReadyCallback*)

This method begins the access-token acquisition procedure. It invokes the provided callback function immediately if the access token is already available and valid.

The function passed into *tokenReadyCallback* should have two parameters of its own:

| Parameter | Type | Description |
| --- | --- | --- |
| *token* | String | The access token |
| *error* | String | Error details, or `null` in the case of success |

#### Example

```squirrel
client.acquireAccessToken(
    function(token, error) {
        if (error) {
            server.error(error);
        } else {
            server.log("The access token has the value: " + token);
        }
    }
);
```

### getValidAccessTokeOrNull()

This method returns an access token string in a non-blocking way. It returns `null` if the client is not authorized or the token has expired.

#### Example

```squirrel
local token = client.getValidAccessTokeOrNull();

if (token) {
    server.log("The access token is valid and has the value: " + token);
} else {
    server.log("The access token has either expired or the client is not authorized");
}
```

### isTokenValid()

This method checks if the access token is valid by comparing its expiry time with current time. It returns a Boolean value: `true` if the token is valid, or `false` if the token has expired.

#### Example

```squirrel
server.log("The access token is " + (client.isTokenValid() ? "valid" : "invalid"));
```

## Complete Example

```squirrel
#require "AWSRequestV4.class.nut:1.0.2"
#require "AWSLambda.agent.lib.nut:1.0.0"
#require "OAuth2.agent.lib.nut:1.0.0

// Substitute with real values
const LAMBDA_REGION        = "us-west-1";
const LAMBDA_ACCESS_KEY_ID = "<YOUR_AWS_ACCESS_KEY_ID>";
const LAMBDA_ACCESS_KEY    = "<YOUR_AWS_ACCESS_KEY>";
const GOOGLE_ISS           = "rsalambda@quick-cacao-168121.iam.gserviceaccount.com";
const GOOGLE_SECRET_KEY    = "-----BEGIN PRIVATE KEY-----\nprivate key goes here\n-----END PRIVATE KEY-----\n";

local signer = AWSLambda(LAMBDA_REGION, LAMBDA_ACCESS_KEY_ID, LAMBDA_ACCESS_KEY);

local providerSettings =  {
    "tokenHost" : "https://www.googleapis.com/oauth2/v4/token"
};

local userSettings = {
    "iss"         : GOOGLE_ISS,
    "jwtSignKey"  : GOOGLE_SECRET_KEY,
    "scope"       : "https://www.googleapis.com/auth/pubsub",
    "rs256signer" : signer
};

local client = OAuth2.JWTProfile.Client(providerSettings, userSettings);

local token = client.getValidAccessTokeOrNull();
if (token != null) {
    // We have a valid token already
    server.log("Valid access token is: " + token);
} else {
    // Acquire a new access token
    client.acquireAccessToken(
        function(newToken, err) {
            if (err) {
                server.error("Token acquisition error: " + err);
            } else {
                server.log("Received a new token: " + newToken);
            }
        }
    );
}
```

**Note** The JSON Web Token (JWT) Profile for OAuth 2.0 was verified and tested with the Google [PubSub](https://cloud.google.com/pubsub/docs/) authorization flow.

## OAuth2.DeviceFlow.Client

This class implements an OAuth 2.0 authorization flow for browserless and/or input-constrained devices. Often referred to as the [device flow](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-05), this flow enables OAuth clients to request user authorization from devices that have an Internet connection but lack a suitable input method or web browser required for a more traditional OAuth flow. This authorization flow therefore instructs the user to perform the authorization request on a secondary device, such as a smartphone.

## OAuth2.DeviceFlow.Client Usage

### constructor(*providerSettings, userSettings*)

This constructor creates an instance of the *OAuth2.DeviceFlow.Client* class. The first parameter, *providerSettings*, is a table that contains provider-specific settings:

| Parameter | Type | Use | Description |
| --- | --- | --- | --- |
| *loginHost* | String | Required | The authorization endpoint. This is used by the client to obtain authorization from the resource owner via user-agent redirection |
| *tokenHost* | String | Required | The token endpoint. This is used by the client to exchange an authorization grant for an access token, typically with client authentication |
| *grantType* | String | Optional. Default: `"urn:ietf:params:oauth:grant-type:device_code"` | The grant type identifier supported by the provider |

The second parameter, *userSettings*, takes a table containing user- and application-specific settings:

| Parameter | Type | Use |Description |
| --- | --- | --- | --- |
| *clientId* | String | Required | The OAuth client ID |
| *clientSecret* | String | Required | The project's client secret |
| *scope* | String | Required | A scope. Scopes enable your application to only request access to the resources that it needs while also enabling users to control the amount of access that they grant to your application |

The library provides predefined configuration settings for the Google Device Auth flow. These settings are defined in the provider-specific settings map: *OAuth2.DeviceFlow.GOOGLE*. This table provides pre-populated *loginHost, tokenHost* and *grantType* values.

#### Device Flow Client Creation Example

```squirrel
local providerSettings =  {
    "loginHost" : "https://accounts.google.com/o/oauth2/device/code",
    "tokenHost" : "https://www.googleapis.com/oauth2/v4/token",
    "grantType" : "http://oauth.net/grantType/device/1.0",
};

local userSettings = {
    "clientId"     : "<USER_FIREBASE_CLIENT_ID>",
    "clientSecret" : "<USER_FIREBASE_CLIENT_SECRET>",
    "scope"        : "email profile",
};

client <- OAuth2.DeviceFlow.Client(providerSettings, userSettings);
```

## OAuth2.DeviceFlow.Client Methods

### acquireAccessToken(*tokenReadyCallback, notifyUserCallback, force*)

This method begins the access-token acquisition procedure. Depending on the client state, it may start a full client authorization procedure or just refresh a token that has already been acquired. It returns `null` in the case of success, or an error message if the client is already performing a request and the *force* directive is set. The access token is delivered through the function passed into the *tokenReadyCallback* function.

Parameter details:

| Parameter | Type | Use | Description |
| --- | --- | --- | --- |
| *tokenReadyCallback* | Function | Required | The handler that will be called when the access token has been acquired, or an error has occurred. The function’s parameters are described below |
| *notifyUserCallback* | Function | Required | The handler that will be called when user action is required. See [RFE, device flow, section 3.3](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-05#section-3.3) for information on what user action might be needed when this callback is executed. The function’s parameters are described below |
| *force* | Boolean | Optional. Default: `false` | This flag forces the token acquisition process to start from the beginning even if a previous request has not yet completed. Any previous session will be terminated |

The *tokenReadyCallback* function should have the following parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| *token* | String | String representation of the access token |
| *error* | String | Error details, or `null` in the case of success |

The *notifyUserCallback* function should have the following parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| *url*  | String | The URL the user needs to use for client authorization |
| *code* | String | The code for the authorization server |

#### Example

```squirrel
client.acquireAccessToken(
    // Token Ready Callback
    function(token, err) {
        if (err) {
            server.error("Token retrieval error: " + err);
        } else {
            server.log("The access token: " + token);
        }
    },
    // User notification callback
    function(url, code) {
        server.log("Authorization is pending. Please grant access.");
        server.log("URL: " + url);
        server.log("CODE: " + code);
    }
);
```

### getValidAccessTokeOrNull()

This method immediately returns either an existing access token if it is valid, or `null` if the token has expired or the client is yet not authorized.

#### Example

```squirrel
local token = client.getValidAccessTokeOrNull();

if (token) {
    server.log("Token is valid: " + token);
} else {
    server.log("Either token expired or client is not authorized!");
}
```

### isTokenValid()

This method checks if the current access token is valid. It returns `true` if this the case, or `false` if the token is no longer valid.

#### Example

```squirrel
server.log("The access token is " + (client.isTokenValid() ? "valid" : "invalid"));
```

### isAuthorized()

This method checks if the client is authorized and able to refresh an expired access token.

#### Example

```squirrel
server.log("The client is " + (client.isAuthorized() ? "authorized" : "unauthorized"));
```

### refreshAccessToken(*tokenReadyCallback*)

This method asynchronously refreshes the access token and invokes the function passed into the *tokenReadyCallback* parameter when this has been completed or an error occurs. The *tokenReadyCallback* function has two parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| *token* | String | The access token |
| *error* | String | Error details, or `null` in the case of success |

#### Example

```squirrel
client.refreshAccessToken(
    function(token, err) {
        if (err) {
            server.error("Token refresh error: " + err);
        } else {
            server.log("The access token is refreshed. It has the value: " + token);
        }
    }
);
```

## Complete Example

```squirrel
#require "OAuth2.agent.lib.nut:1.0.0

// Fill CLIENT_ID and CLIENT_SECRET with correct values
local userConfig = {
    "clientId"     : "<CLIENT_ID>",
    "clientSecret" : "<CLIENT_SECRET>",
    "scope"        : "email profile",
};

// Initialize client with provided Google Firebase config
client <- OAuth2.DeviceFlow.Client(OAuth2.DeviceFlow.GOOGLE, userConfig);

local token = client.getValidAccessTokeOrNull();

if (token != null) {
    server.log("Valid access token is: " + token);
} else {
    // Acquire a new access token
    local error = client.acquireAccessToken(
        // Token received callback function
        function(resp, err) {
            if (err) {
                server.error("Token acquisition error: " + err);
            } else {
                server.log("Received token: " + resp);
            }
        },
        // User notification callback function
        function(url, code) {
            server.log("Authorization is pending. Please grant access.");
            server.log("URL: " + url);
            server.log("CODE: " + code);
        }
    );
    
    if (error != null) server.error("Client is already performing request (" + error + ")");
}
```

**Note** The DeviceFlow Client was verified and tested using the Google [Firebase](https://firebase.google.com) authorization flow.

# License

The OAuth library is licensed under the [MIT License](LICENSE).
