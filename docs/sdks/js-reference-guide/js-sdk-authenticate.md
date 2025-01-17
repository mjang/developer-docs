---
title: authenticate
id: js-reference-authenticate
description: ""
slug: /js-reference-authenticate
keywords:
  - javascript sdk
pagination_next: null
pagination_prev: null
last_update:
  date: 08/01/2023
  author: William May
draft: false
doc_type: reference
displayed_sidebar: sdkSidebar
---

The **authenticate** function enables an app using the Beyond Identity Javascript SDK to perform passkey-based authentication within a standard OpenID Connect authorization flow.

:::note
_The [example app](https://github.com/gobeyondidentity/bi-sdk-js/tree/main/example) for the Javascript SDK uses [NextAuth.js](https://next-auth.js.org/getting-started/example) to initiate the OIDC flow and to consume the resulting code and token._  
:::

## Dependencies

The **authenticate** function requires the Beyond Identity Javascript SDK.

```bash
npm install @beyondidentity/bi-sdk-js
```

## Prerequisites

Before making a call to **authenticate**, you must complete the following prerequisite calls:

1. Import the required types and functions from the SDK.

  ```javascript
  import { Embedded } from "@beyondidentity/bi-sdk-js";
  ```

1. Initialize the SDK.

  ```javascript
  const embedded = await Embedded.initialize();
  ```

1. Identify the passkey you wish to authenticate with and obtain its passkey ID.

  > _How you achieve this depends upon your app, but you can obtain a list of passkeys available on the device via the **getPasskeys** function. This returns an array of passkeys that you can use, for example, to prompt the user interactively to select one. The **id** property of the selected **Passkey** is the passkey id this function expects_

4. Use isAuthenticateUrl to verify the `url` parameter you intend to send to the function

  ```javascript
  await embedded.isAuthenticateUrl(url);
  ```

## Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| **url** | string | Required. A Beyond Identity authentication URL is generated by the Beyond Identity API's `/authorize` endpoint in response to a standard OpenID Connect request from your app (see [example](#example:-retrieve-beyond-identity-authentication-url-via-oidc-call) below). The generated URL is unique for each authentication request. It contains an encoded JWT token containing the challenge for the passkey to sign. |
| **passkeyId** | string | The ID of the passkey that you wish to use for the authentication. This should match the **id** property of a **Passkey** available on the device. |

## Returns

On success, the **authenticate** function returns a Promise that resolves to an **AuthenticateResponse**, which itself is a JSON object that contains the following keys:

- **redirectURL**: string containing the complete URL to which your app should redirect the user to complete the OIDC flow.

  Keeping with the OIDC specifications, this includes the code and state parameters as query parameters to the redirect_url specified in the original OIDC request to the `/authorize` endpoint for the authentication URL.

- **message**: string containing a message your app may optionally consume or display.

  On success, the **authenticate** function returns a Promise that resolves to an **AuthenticateResponse**, which itself is a JSON object that contains the following keys:

- **redirectURL**: string containing the complete URL to which your app should redirect the user to complete the OIDC flow. Keeping with the OIDC specifications, this includes the code and state parameters as query parameters to the redirect_url specified in the original OIDC request to the `/authorize` endpoint for the authentication URL.

- **message**: string containing a message your app may optionally consume or display.

## Notes

Using the **authenticate** function requires your app to generate a standard OpenID Connect (OIDC) request to Beyond Identity's API and consume the resulting codes and tokens.

> The [example app](https://github.com/gobeyondidentity/bi-sdk-js/tree/main/example) for the Javascript SDK uses [NextAuth.js](https://next-auth.js.org/getting-started/example) to initiate the OIDC flow and to consume the resulting code and token.

For step-by-step instructions to configure NextAuth.js and OIDC using our sample application, and to create the associated Beyond Identity tenant configuration, see [Getting Started](/docs/next/get-started). For complete guidance on authentication, see <mark>Authentication with Passkey (content has changed; will need a new link)</mark>

## Examples

### Example: Call **authenticate** after validating URL

```javascript
if (embedded.isAuthenticateUrl(url)) {
  authenticateResponse = await embedded.authenticate(url, passkeyId);
}
```

### Example: Call **authenticate** with selected ID after prompting the user with a list of passkeys

```javascript
let passkeys = await embedded.getPasskeys();
let promptText = passkeys.map((passkey, index) => {
    return `${index}: ${passkey.identity.username}`;
}).join("\n");
let selectedIndex = parseInt(prompt(promptText, "index")!!);
if (selectedIndex >= 0 && selectedIndex < passkeys.length) {
    let selectedId = passkeys[selectedIndex].id;
    let result = await embedded.authenticate(url, selectedId);
}
```

### Example: Retrieve Beyond Identity authentication url via OIDC call

The app sends an OIDC call to the Beyond Identity API's `/authorize` endpoint:

```http
GET https://auth-us.beyondidentity.com/v1/tenants/{TENANT_ID}/realms/{REALM_ID}/applications/{APPLICATION_ID}/authorize?client_id={CLIENT_ID}&scope=openid&response_type=code&redirect_uri={REDIRECT_URI}&state=8LIY29kN8Oz7zrAhb8xb0yvem-gvnRy1HTn03MAuL_E
```

where the following elements match the corresponding properties of the app as configured in your Beyond Identity tenant:

| Property | Description |
| --- | --- |
| **TENANT_ID** | The Tenant ID of the tenant in which the app is configured. |
| **REALM_ID** | The Realm ID of the realm in which the app is configured. |
| **APPLICATION_ID** | The Application ID from the header of the app's configuration page. |
| **CLIENT_ID** | The Client ID from the External Protocol tab of the app's configuration page. |
| **REDIRECT_URI** | Matches one of the Redirect URIs configured on the External Protocol tab of the app's configuration page, URL encoded. |

When the Invocation Type configured on the Authenticator Config tab of the app's configuration page is set to Manual, it returns a JSON object:

```json
{ "authenticate_url": "http://localhost:8083/bi-authenticate?request={BI_JWT}" }
```

where **BI_JWT** is a base64url encoded JWT token containing the challenge and other data to kick off the passkey authentication.

When the Invocation Type on the app is set to Automatic, it returns an HTTP 302 to the authentication URL:

```http
http/1.1 302 Found
...
location: http://localhost:8083/bi-authenticate?request={BI_JWT}
```

where **BI_JWT** is a base64url encoded JWT token containing the challenge and other data to kick off the passkey authentication.
