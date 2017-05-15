# Demo Instructions

This example shows how to set up your Google account to use Firebase authorization. 

## Adding credentials

1. Go to `https://console.developers.google.com` and log in
1. Select `Credentials` in the left bar
1. Click on the `Create credentials` button
1. Select `OAuth client ID`
1. Select `Other`
1. Enter a name and press `Create`
1. Copy client ID and client secret from popup window shown in the browser

**NOTE**: If you have lost your Client ID and Secret click on the ID name in the `OAuth 2.0 client IDs` list and copy them from the Client ID details page.

## Customizing consent screen

To customize the page that users see while authorizing your application go to `OAuth consent screen` tab.


## Setting up Agent Code

Copy the example agent [code](Firebase.agent.nut) and paste it into the Web IDE to the left window (agent code).

Set the example code configuration parameters with values retrieved on the previous steps:

Parameter             | Description
----------------------| -----------
CLIENT_ID  			  | Firebase client ID
CLIENT_SECRET         | Firebase client secret

**NOTE**: As the sample code includes the private key verbatim in the source,
it should be treated carefully, and not checked into version control!

# License

The OAuth library is licensed under the [MIT License](../LICENSE).
