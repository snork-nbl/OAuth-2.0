# OAuth2 DeviceFlow Client

This library implements the OAuth2 authorization flow for for browserless and input constrained devices. The implementation conforms to limited device authorization [spec](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-05).

Basic functionality consists of following actions:

- Token acquisition
- Checking if token is valid
- Checking if the application is authorized
- Refresh operation for expired access token

The Client is tested and verified on the Google 
[Firebase](https://firebase.google.com) authorization flow.

**To add this library to your project, add** `#require "OAuth2.DeviceFlow.agent.lib.nut:1.0.0"` **to the top of your agent code**


## OAuth2.DeviceFlow.Client public methods

### constructor(providerSettings, userSettings)

Construction that creates an instance of the OAuth2 Client. 

The first parameter `providerSettings` is a map that contains provider specific settings:
 
| Parameter | Type | Default Value | Description |
| --- | --- | --- | --- |
| `LOGIN_HOST` | string | mandatory field |Provider's authorization server  |
| `TOKEN_HOST` | string | mandatory field | URL of the endpoint to poll for authorization response |
| `GRANT_TYPE` | string | `urn:ietf:params:oauth:grant-type:device_code` |Grant type value supported by the provider |

The second parameter `userSettings` defines a map with user and application specific settings:

| Parameter | Type | Description |
| --- | --- | --- |
| `clientId` | string | OAuth client ID |
| `clientSecret` | string | The project's client secret |
| `scope` | string | Scopes enable your application to only request access to the resources that it needs while also enabling users to control the amount of access that they grant to your application. |

The library provides predefined configuration settings for 
Google Device Auth flow. These settings are defined in the provider 
specific settings map:`OAuth2.DeviceFlow.GOOGLE`. The table
provides `LOGIN_HOST`, `TOKEN_HOST` and `GRANT_TYPE` values. 

#### Example

```squirrel
    providerSettings =  {
        "LOGIN_HOST" : "https://accounts.google.com/o/oauth2/device/code",
        "TOKEN_HOST" : "https://www.googleapis.com/oauth2/v4/token",
        "GRANT_TYPE" : "http://oauth.net/grant_type/device/1.0",
    };
    userSettings = {
        "clientId"     : "USER_FIREBASE_CLIENT_ID",
        "clientSecret" : "USER_FIREBASE_CLIENT_SECRET",
        "scope"        : "email profile",
    };

    client <- OAuth2.DeviceFlow.Client(providerSettings, userSettings);
```

### acquireAccessToken(tokenReadyCallback, notifyUserCallback, force)

Starts access token acquisition procedure. Depending on Client state may starts full client authorization procedure or
just token refreshing. Returns null in case of success and error otherwise.

Parameter details:

| Parameter | Type | Description |
| --- | --- | --- |
| *tokenReadyCallback* | Function | The handler to be called when access token is acquired or error is observed |
| *notifyUserCallback* | Function | The handler to be called when user action is required. See [RFE, device flow, section3.3](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-05#section-3.3) |
| *[force]* | Boolean | [optional] the directive to start new acquisition procedure even if previous request is not complete. Default value is `false` |

`tokenReadyCallback` parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| token | String | String representation of access token |
| error | Table | Table with  error details, `null` in case of success |


`notifyUserCallback` parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| uri | String | the URI the user need to use for client authorization |
| code | String | the code the user need to use somewhere at authorization server |

#### Example

Using `client` from previous sample

```squirrel
client.acquireAccessToken(
    function(resp, err) {
        server.log(resp);
        if (err) {
            server.error(err);
        }
    },
    function(url, code) {
        server.log("Authorization is pending. Please grant access.");
        server.log("URL: " + url);
        server.log("CODE: " + code);
    }
);
```
### getValidAccessTokeOrNull()

Returns access token string non blocking way. Returns access token as a string object if token is valid, null if the client is not authorized or token is expired.

#### Example

Using `client` from the first sample

```squirrel
local token = client.getValidAccessTokeOrNull();
if (token) server.log("token is valid and has value: " + token);
else server.log("token is either expired  or client is not authorized!");
```

### isTokenValid()

Checks if access token is valid.

#### Example

Using `client` from the first sample

```squirrel
server.log("token is valid=" + client.isTokenValid());
```

### isAuthorized()

Checks if the client is authorized and able to refresh expired access token.

Using `client` from the first sample

```squirrel
server.log("client is authorized=" + client.isAuthorized());
```


### refreshAccessToken(tokenReadyCallback)

Refreshes access token non blocking way, will invoke `tokenReadyCallback` in case of success.

`tokenReadyCallback` parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| token | String | String representation of access token |
| error | Table | Table with  error details, `null` in case of success |

#### Example

Using `client` from the first sample

```squirrel
client.refreshAccessToken(
    function(resp, err) {
        server.log(resp);
        if (err) {
            server.error(err);
        }
    }
);
```

## Complete usage sample

To connect all the parts together and show a sample of common case of library usage let's take a look a following sample

```squirrel
#require "OAuth2.DeviceFlow.agent.lib.nut:1.0.0

// Fill CLIENT_ID and CLIENT_SECRET with correct values
local userConfig = {
    "clientId"     : "CLIENT_ID",
    "clientSecret" : "CLIENT_SECRET",
    "scope"        : "email profile",
};

// Initializing client with provided Google Firebase config
client <- OAuth2.DeviceFlow.Client(OAuth2.DeviceFlow.GOOGLE, userConfig);

local token = client.getValidAccessTokeOrNull();
if (token != null) {
    server.log("Valid access token is: " + token);
} else {
    // Starting procedure of access token acquisition
    local error = client.acquireAccessToken(
        function(resp, err) {
            if (err) {
                server.error("Token acquisition error: " + err);
            } else {
                server.log("Received token: " + resp);
            }
        },
        function(url, code) {
            server.log("Authorization is pending. Please grant access.");
            server.log("URL: " + url);
            server.log("CODE: " + code);
        }
    );

    if (null != error) server.error("Failed to obtain token: " + error);
}
```

# License

The OAuth library is licensed under the [MIT License](LICENSE).