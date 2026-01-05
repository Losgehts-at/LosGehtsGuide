---
title: OAuth 2.0 Integration Guide  
---
This document describes how to integrate with our OAuth 2.0 API for authentication and user data retrieval. It covers the full flow: authorization, token exchange, and user information retrieval. This guide is framework-agnostic and can be applied in web, mobile, or server-side applications.

## Base URLs

**Development Environment:**

```
https://dev.losgehts.at/
```

**Production Environment:**

```
https://ident.losgehts.at/
```

## 1. Overview

OAuth 2.0 allows your application to access user data securely without handling credentials directly. The typical flow involves:

1. Redirecting the user to the authorization endpoint.
2. Receiving an authorization code.
3. Exchanging the code for an access token.
4. Using the token to retrieve user information.

## 2. Authorization Endpoint

**URL:**

```
GET /oauth/authorize
```

### Query Parameters

| Parameter  | Required | Description |
|------------|----------|-------------|
| client_id  | Yes      | The unique client identifier of your application. |
| state      | Yes      | A unique string used to maintain state between the request and callback (e.g., session ID, affiliate tag, or encrypted UserID). Returned in the callback to help prevent CSRF attacks. |
| scope      | Yes      | A space-separated list of scopes defining the permissions being requested.. Supported scopes include: `signup`, `kyc`, and `sof`. |
| locale     | No       | A two-letter country/language code used for localization (e.g., `de`, `at`, `us`). Primarily determines the UI language. |
| country         | No       | Enforces a specific country context in a multi-country setup (e.g., `AT`). If not provided, the country will be auto-detected based on the user's geolocation. |


### Example

**Development:**

```
https://dev.losgehts.at/oauth/authorize?client_id=40&state=abc123&scope=signup&locale=de
```

**Production:**

```
https://ident.losgehts.at/oauth/authorize?client_id=40&state=abc123&scope=signup&locale=de
```
**Security Note:** Always validate the `state` parameter in the callback.

### Try It Out: OAuth Generator

Want to generate authorization URLs without writing any code? Use our **interactive OAuth Generator**:

- Enter your **Client ID**, **Client Secret**, preferred **Language**, **Market**, and **Scope**.
- Click **Generate URL** to see the exact `/oauth/authorize` URL.
- Click **Continue to Authorization** to go through the OAuth flow and view the response in real-time.
- Inspect the returned **access token** and **user information** safely in the sandbox.

[Open OAuth Demo →](https://demo.losgehts.at/)

## 3. Callback Handling

After authorization, the user is redirected to your callback URL with:

* `code` — The authorization code
* `state` — The same value you supplied

**Example callback URL:**

```
https://yourapp.com/callback?code=AUTH_CODE&state=abc123
```

## 4. Token Exchange

Exchange the authorization code for an access token.

**Endpoint:**

```
POST /oauth/token
Content-Type: application/json
```

### Request Body

```json
{
  "code": "AUTHORIZATION_CODE",
  "state": "STATE_VALUE",
  "client_id": "CLIENT_ID",
  "client_secret": "CLIENT_SECRET",
  "grant_type": "authorization_code"
}
```

### Response Example

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "xyz123"
}
```

**Important:** Store the access token securely and never expose it in client-side code.

## 5. User Information Endpoint

Use the access token to retrieve user information.

**Endpoint:**

```
POST /oauth/userinfo
Content-Type: application/json
```

### Request Body

```json
{
  "token": "ACCESS_TOKEN",
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET"
}
```

### Response Example

```json
{
  "Verification": {
    "VerificationId": 354,
    "OAuthState": "55jhg8l1t53",
    "OAuthScope": "signup email phonenumber",
    "Password": "asdsad2132132",
    "Email": "demo@losgehts.at",
    "EmailConfirmed": 1,
    "VerificationStatusId": 2,
    "FirstName": "John",
    "FirstNameVerified": 1,
    "LastName": "Doe",
    "LastNameVerified": 1,
    "DateOfBirth": "1990-12-31",
    "DateOfBirthVerified": 1,
    "Gender": null,
    "GenderVerified": 0,
    "Nationality": null,
    "NationalityVerified": 0,
    "Street": "Feldstraße",
    "StreetVerified": 1,
    "HouseNumber": "12",
    "HouseNumberVerified": 1,
    "ZipCode": "4020",
    "ZipCodeVerified": 1,
    "Town": "Linz",
    "TownVerified": 1,
    "Country": "AT",
    "CountryVerified": 1,
    "Region": "Oberösterreich",
    "RegionCode": "4",
    "PhoneNumber": "436604737427",
    "PhoneNumberInternational": "436604737427",
    "PhoneNumberNational": "0660 4737427",
    "PhoneCountryCode": "AT",
    "PhoneCountryPrefix": "43",
    "PhoneNumberVerified": 0,
    "PhoneNumberConfirmed": null,
    "Iban": null,
    "IbanVerified": 0,
    "Lang": "DE",
    "Currency": "EUR",
    "LimitAmount": null,
    "DepositAmount": 10,
    "MarketingOptIn": 1,
    "AcceptedPrivacy": 1,
    "AcceptedTerms": 1
  },
  "AuditLog": []
}
```

### Verification Status Values

The `verificationStatus` field can contain the following values:

| Status ID | Status  | Description                                           |
| --------- | ------- | ----------------------------------------------------- |
| 1         | Pending | Verification process is still in progress             |
| 2         | Full    | Complete verification has been successfully completed |
| 3         | Passive | Passive verification has been completed               |
| 4         | Failed  | Verification process has failed                       |

### Complete Response Fields Reference

All fields listed below are returned inside Verification. All verification fields must be accessed via `response.Verification`

|           Field          |   Type  |                   Description                  |                   Format                   |
|:------------------------:|:-------:|:----------------------------------------------:|:------------------------------------------:|
| VerificationId           | Integer | Primary identifier for the verification record |                                            |
| OAuthState               | Text    | OAuth state for authentication                 |                                            |
| OAuthScope               | Text    | OAuth scopes granted                           | Scopes added automatically per config      |
| Email                    | Text    | User's email address                           |                                            |
| EmailConfirmed           | Boolean | Whether email has been confirmed               |                                            |
| VerificationStatusId     | Integer | Status of the verification process             |                                            |
| Password                 | Text    | User's password (encrypted)                    |                                            |
| FirstName                | Text    | User's first name                              |                                            |
| FirstNameVerified        | Boolean | Whether first name has been verified           |                                            |
| LastName                 | Text    | User's last name                               |                                            |
| LastNameVerified         | Boolean | Whether last name has been verified            |                                            |
| DateOfBirth              | Text    | User's date of birth                           | ISO 8601 calendar date format (YYYY-MM-DD) |
| DateOfBirthVerified      | Boolean | Whether date of birth has been verified        |                                            |
| Gender                   | Text    | User's gender                                  | MALE, FEMALE, OTHER                        |
| GenderVerified           | Boolean | Whether gender has been verified               |                                            |
| Nationality              | Text    | User's nationality                             | ISO 3166-1 alpha-2 (two uppercase letters) |
| NationalityVerified      | Boolean | Whether nationality has been verified          |                                            |
| ZipCode                  | Text    | User's postal/zip code                         |                                            |
| ZipCodeVerified          | Boolean | Whether zip code has been verified             |                                            |
| Town                     | Text    | User's town/city                               |                                            |
| TownVerified             | Boolean | Whether town has been verified                 |                                            |
| Street                   | Text    | User's street name                             |                                            |
| StreetVerified           | Boolean | Whether street has been verified               |                                            |
| HouseNumber              | Text    | User's house number                            |                                            |
| HouseNumberVerified      | Boolean | Whether house number has been verified         |                                            |
| Country                  | Text    | User's country                                 | ISO 3166-1 alpha-2 (two uppercase letters) |
| CountryVerified          | Boolean | Whether country has been verified              |                                            |
| Region                   | Text    | User's Region Name                             |                                            |
| RegionCode               | Text    | User's Region                                  | Region code (country-specific)             |
| Iban                     | Text    | User's IBAN                                    |                                            |
| IbanVerified             | Boolean | Whether IBAN has been verified                 |                                            |
| PhoneNumber              | Text    | User's phone number                            | E.164 standard (436803104850)              |
| PhoneNumberConfirmed     | Boolean | Whether phone number has been confirmed        |                                            |
| PhoneNumberInternational | Text    | User's international phone number              | E.164 standard (436803104850)              |
| PhoneNumberNational      | Text    | User's national phone number                   | e.g. (0680 3104850)                        |
| PhoneCountryCode         | Text    | Country calling prefix                         | ISO 3166-1 alpha-2 (two uppercase letters) |
| PhoneCountryPrefix       | Text    | Phone country prefix                           | Without "+" e.g. "43"                      |
| PhoneNumberVerified      | Boolean | Whether phone number has been verified         |                                            |
| Lang                     | Text    | User's language preference                     | ISO 3166-1 alpha-2 (two uppercase letters) |
| Currency                 | Text    | User's preferred currency                      | ISO 4217 (three uppercase letters)         |
| LimitAmount              | Numeric | User's limit amount                            |                                            |
| DepositAmount            | Numeric | User's deposit amount                          |                                            |
| MarketingOptIn           | Boolean | Whether user opted in to marketing             |                                            |
| AcceptedPrivacy          | Boolean | Whether user accepted privacy policy           |                                            |
| AcceptedTerms            | Boolean | Whether user accepted terms                    |                                            |

## AuditLog

The `AuditLog` field provides a chronological list of audit events recorded during the verification process.  
It can be used to trace which user data was captured and when.

### Format

`AuditLog` is returned as a **JSON array** of audit log entries.

Each entry represents a single event in the verification lifecycle.

### AuditLog Entry Fields

| Field                        | Type               | Description |
|-----------------------------|--------------------|-------------|
| AuditLogId                  | Integer            | Unique identifier of the audit log entry |
| CreatedAt                   | Text               | UTC timestamp of the event (ISO 8601) |
| VerificationId              | Integer            | Related verification identifier |
| VerificationProviderId      | Integer \| Null    | Verification provider reference, if applicable |
| VerificationProviderTableId | Integer \| Null    | Provider-specific table reference |
| AuditMessage                | Text               | Human-readable description of the event |
| AuditData                   | Text \| Null       | Event-specific data (see below) |

### AuditData Field

The `AuditData` field may contain:

- A **plain string** (e.g. email address)
- A **JSON-encoded string** containing structured data
- `null` when no additional data is stored (e.g. password events)

When `AuditData` contains JSON, clients must **parse the string as JSON**.

### Example

```json
{
  "AuditLog": [
    {
      "AuditLogId": 50,
      "CreatedAt": "2025-12-28T21:13:06.452Z",
      "VerificationId": 196,
      "VerificationProviderId": null,
      "VerificationProviderTableId": null,
      "AuditMessage": "Email captured",
      "AuditData": "user@example.com"
    },
    {
      "AuditLogId": 52,
      "CreatedAt": "2025-12-28T21:13:12.621Z",
      "VerificationId": 196,
      "VerificationProviderId": null,
      "VerificationProviderTableId": null,
      "AuditMessage": "Phone captured",
      "AuditData": "{\"PhoneNumber\":\"436801114120\",\"PhoneCountryCode\":\"AT\"}"
    }
  ]
}
```

**Notes:**

* Fields ending in 'Verified' indicate that the value has been confirmed.
* Some fields are only returned if the requested scopes include them.

## 6. Handling the `state` Parameter

1. Generate a unique state per OAuth request.
2. Include it in the `/oauth/authorize` redirect.
3. Validate that the returned state matches the original before exchanging the code.

This protects against CSRF attacks.

## 7. Typical OAuth Flow

1. **User Authorization** → Redirect to `/oauth/authorize`
2. **Receive Code** → Extract code and state from callback
3. **Token Exchange** → POST to `/oauth/token`
4. **Fetch User Info** → POST to `/oauth/userinfo`
5. **Store & Use Data** → Store tokens securely

## 8. Security Considerations

* Always validate the state parameter
* Use HTTPS for all OAuth traffic
* Never expose `client_secret` in browsers or JS
* Store tokens securely (server-side or encrypted local storage for mobile)
