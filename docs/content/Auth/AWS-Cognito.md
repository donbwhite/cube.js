---
title: AWS Cognito Guide
permalink: /security/jwt/aws-cognito
category: Authentication & Authorization
subCategory: Guides
menuOrder: 4
---

## Introduction

In this guide, you'll learn how to integrate AWS Cognito authentication with a
Cube.js deployment. If you already have a pre-existing Cognito User Pool in AWS
that you'd like to re-use, please skip ahead to
[Configure Cube.js](#configure-cube-js).

## Create and configure a User Pool

If you haven't already created a User Pool, please follow [the instructions in
the AWS Cognito documentation][link-aws-cognito-hosted-ui] to create one, along
with enabling the Hosted UI.

### Custom claims

To add custom claims to the JWT, you will need to associate [a Lambda
function][link-aws-lambda] to the [Pre Token Generation event
trigger][link-aws-cognito-pre-token] available on your User Pool.

First, create a Lambda function in the AWS Lambda Console using the snippet
below:

```javascript
exports.handler = (event, context, callback) => {
  event.response = {
    claimsOverrideDetails: {
      claimsToAddOrOverride: {
        'http://localhost:4000/': JSON.stringify({
          company_id: 'company1',
          user_id: event.request.userAttributes.sub,
          roles: ['user'],
        }),
      },
    },
  };
  callback(null, event);
};
```

Then navigate to the Amazon Cognito Console and select Triggers from the left
sidebar, then associate the Lambda function you created previously.

You can find more examples of [modifying claims in JWTs
here][link-aws-cognito-pretoken-example].

## Configure Cube.js

Now we're ready to configure Cube.js to use AWS Cognito. Go to your Cube.js
project and open the `.env` file and add the following, replacing the values
wrapped in `<>`.

```dotenv
CUBEJS_JWK_URL=https://cognito-idp.<AWS_REGION>.amazonaws.com/<USER_POOL_ID>/.well-known/jwks.json
CUBEJS_JWT_AUDIENCE=<APPLICATION_URL>
CUBEJS_JWT_ISSUER=https://cognito-idp.<AWS_REGION>.amazonaws.com/<USER_POOL_ID>
CUBEJS_JWT_ALGS=RS256
CUBEJS_JWT_CLAIMS_NAMESPACE=<CLAIMS_NAMESPACE>
```

## Testing with the Developer Playground

### Retrieving a JWT

Go to the [OpenID Playground from Auth0][link-openid-playground] to and click
Configuration.

<p
  style="text-align: center"
>
  <img
  src="https://raw.githubusercontent.com/cube-js/cube.js/master/docs/content/Auth/auth0-03-get-jwt-01.png"
  style="border: none"
  width="80%"
  />
</p>

Change the Server Template to Custom, and enter the following values:

<p
  style="text-align: center"
>
  <img
  src="https://raw.githubusercontent.com/cube-js/cube.js/master/docs/content/Auth/cognito-get-jwt-02.png"
  style="border: none"
  width="80%"
  />
</p>

- **Discovery Document URL**:
  `https://cognito-idp.<AWS_REGION>.amazonaws.com/<USER_POOL_ID>/.well-known/openid-configuration`
- **OIDC Client ID**: Retrieve from App Client settings page in AWS Cognito
  Console
- **OIDC Client Secret**: Retrieve from App Client settings page in AWS Cognito
  Console

Click 'Use Discovery Document' to auto-fill the remaining values, then click
Save.

<!-- prettier-ignore-start -->
[[warning |]]
| If you haven't already, go back to the AWS Cognito App Client's settings and
| add `https://openidconnect.net/callback` to the list of allowed callback
| URLs.
<!-- prettier-ignore-end -->

Now click Start; and in a separate tab, go to the App Client's settings page and
click the Launch Hosted UI button.

<p
  style="text-align: center"
>
  <img
  src="https://raw.githubusercontent.com/cube-js/cube.js/master/docs/content/Auth/cognito-get-jwt-03.png"
  style="border: none"
  width="80%"
  />
</p>

If the login is successful, you should be redirected to the OpenID Connect
Playground. Click on the Exchange button to exchange the code for your tokens:

<p
  style="text-align: center"
>
  <img
  src="https://raw.githubusercontent.com/cube-js/cube.js/master/docs/content/Auth/cognito-get-jwt-04.png"
  style="border: none"
  width="80%"
  />
</p>

Click Next, and continue on to the next section and click the Verify button to
verify the JWT signature as well as decode the identity token:

<p
  style="text-align: center"
>
  <img
  src="https://raw.githubusercontent.com/cube-js/cube.js/master/docs/content/Auth/cognito-get-jwt-05.png"
  style="border: none"
  width="80%"
  />
</p>

### Set JWT in Developer Playground

Now open the Developer Playground (at `http://localhost:4000`) and on the Build
page, click Add Security Context.

<p
  style="text-align: center"
>
  <img
  src="https://raw.githubusercontent.com/cube-js/cube.js/master/docs/content/Auth/auth0-04-dev-playground-01.png"
  style="border: none"
  width="80%"
  />
</p>

Click the Token tab, paste the `id_token` from OpenID Playground and click the
Save button.

<p
  style="text-align: center"
>
  <img
  src="https://raw.githubusercontent.com/cube-js/cube.js/master/docs/content/Auth/auth0-04-dev-playground-02.png"
  style="border: none"
  width="80%"
  />
</p>

Close the popup and use the Developer Playground to make a request. Any schemas
using the [Security Context][ref-sec-ctx] should now work as expected.

[link-aws-cognito-hosted-ui]:
  https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-app-integration.html#cognito-user-pools-create-an-app-integration
[link-aws-cognito-pre-token]:
  https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-pre-token-generation.html
[link-aws-cognito-pretoken-example]:
  https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-pre-token-generation.html#aws-lambda-triggers-pre-token-generation-example-1
[link-aws-lambda]: https://docs.aws.amazon.com/lambda/latest/dg/welcome.html
[link-openid-playground]: https://openidconnect.net/
[ref-sec-ctx]: /security/context
