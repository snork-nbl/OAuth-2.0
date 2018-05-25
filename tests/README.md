# Test Instructions

## Google JWT profile tests (PubSub scope)

Follow the JWT example [instructions](examples#jwt-profile-for-oauth-20) to obtain the `GOOGLE_ISS` and `GOOGLE_SECRET_KEY` variables and set them as the environment variables. This test suite can be run automatically:

```
impt test run --tests GooglePubSubJWTAuth.agent.test.nut::
```

## Google Device Flow tests

Follow the OAuth 2.0 Device Flow example [instructions](examples#oauth-20-device-flow) to obtain the `CLIENT_ID` and `CLIENT_SECRET` variables and set them as the environment variables. This test requires some manual interaction and can be run:

```
impt test run --tests GoogleDeviceFlow.agent.test.nut::
```