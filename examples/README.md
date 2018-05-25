# Demo Instructions

* [JWT Profile for OAuth 2.0](#jwt-profile-for-oauth-20)
* [OAuth 2.0 Device Flow](#oauth-20-device-flow)

## JWT Profile for OAuth 2.0

The examples shows how to acquire access token from Google
 [OAuth service](https://developers.google.com/identity/protocols/OAuth2)
 via Google [OAuth 2.0 for Service Accounts](https://developers.google.com/identity/protocols/OAuth2ServiceAccount)
 Protocol, which implements the `JWT Profile for OAuth 2.0`
 [specification](https://tools.ietf.org/html/rfc7523).

### Setting up Google OAuth2 for Service Accounts

Google [`OAuth2 for Service Accounts`](https://developers.google.com/identity/protocols/OAuth2ServiceAccount)
 implement the `JWT Profile for OAuth 2.0` [specification](https://tools.ietf.org/html/rfc7523).
 To obtain the Google account's credentials follow the steps
 (instructions below assume that you are registered at https://console.cloud.google.com):
1. Open and login at [`Google cloud console`](https://console.cloud.google.com/projectselector/home/dashboard).
1. If you have an existing project that you want to work with, skip to the next step.
Otherwise click on the project selector (the link to the right from the `Google Cloud Platform`
icon on the top right corner of the screen) and press `+` in the opened window.
1. Select the required project in the project selector (the link to the right from the
`Google Cloud Platform` icon on the top right corner of the screen).
1. Click `IAM & Admin`, then `Service Accounts` from left side menu.
1. Press `Create service account` button.
1. Provide new service account name at corresponding field.
1. From drop down menu `Role` select new account role.
For current example select all available `Pub/Sub` group roles.
1. Check `Furnish a new private key` button. Leave other checkboxes untouched.
1. Press `Create` button.
1. If no key is generated (`Key ID` field contains `No key`), create new public/private key pair
by pressing `Create key` from right drop down menu of the selected service account.
Select `JSON` at popup window and press `CREATE`.
1. The file `<project name>-<random number>.json` will be downloaded to your machine.

An example of downloaded file:
``` JSON
{
  "type": "service_account",
  "project_id": "test-project",
  "private_key_id": "27ed751da7f0cb605c02dafda6a5cf535e662aea",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMII ..... QbDgw==\n-----END PRIVATEKEY-----\n",
  "client_email": "test-account@test-project.iam.gserviceaccount.com",
  "client_id": "117467107027610486288",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://accounts.google.com/o/oauth2/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/test-account%40@test-project.iam.gserviceaccount.com"
}
```

### Setting up Agent Code

Here is some agent [code](JWTProfile.agent.nut).
Copy and paste it into the Web IDE to the left window (agent code).

Set the example code configuration parameters with values retrieved on the previous steps:

Parameter             | Description
----------------------| -----------
GOOGLE_ISS            | Use `client_email` field from [downloaded](#setting-up-google-oauth2-for-service-accounts) JSON file
GOOGLE_SECRET_KEY     | Use `private_key` field from [downloaded](#setting-up-google-oauth2-for-service-accounts) JSON file

Run the example code and it should print acquired access token.

## OAuth 2.0 Device Flow

This example demonstrates acquiring access token from
[OAuth service](https://developers.google.com/identity/protocols/OAuth2) using
[OAuth 2.0 for TV and Limited-Input Device Applications](https://developers.google.com/identity/protocols/OAuth2ForDevices).

### Creating Google Client Credentials

Instructions below assume that you are registered at https://console.cloud.google.com.

1. Open and log in at [`Google cloud console`](https://console.cloud.google.com/projectselector/home/dashboard).
1. Select the required project in the project selector (the link to the right from the
`Google Cloud Platform` icon on the top right corner of the screen).
1. Select `APIs and Services` from the left side menu.
1. Select `Credentials` in the left bar.
1. Go to the `OAuth consent screen` tab and type in your public product name into
the `Product name shown to users` field. Press `Save`.
1. Select the `Credentials` tab.
1. Click on the `Create credentials` button.
1. Select `OAuth client ID`.
1. Select `Other`.
1. Enter a name and press `Create`
1. Copy client ID and client secret from popup window shown in the browser

**NOTE**: If you have lost your Client ID and Secret click on the ID name in the `OAuth 2.0 client IDs`
list and copy them from the Client ID details page.

### Customizing Consent Screen

To customize the page that users see while authorizing your application go to
`OAuth consent screen` tab.

### Setting up Agent Code

Copy the example agent [code](DeviceFlow.agent.nut) and paste it into the Web IDE to the left
window (agent code).

Set the example code configuration parameters with values retrieved on the previous steps:

Parameter             | Description
----------------------| -----------
CLIENT_ID  			      | Google [Client ID](#creating-google-client-credentials)
CLIENT_SECRET         | Google [Client Secret](#creating-google-client-credentials)

**NOTE**: As the sample code includes the private key verbatim in the source,
it should be treated carefully, and not checked into version control!

# License

The OAuth library is licensed under the [MIT License](../LICENSE).
