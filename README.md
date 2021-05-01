[![Build Status](https://travis-ci.org/Relesi/ponto-inteligente-api-1.1.svg?branch=master)](https://travis-ci.org/Relesi/ponto-inteligente-api-1.1)


# OAuth2 
This repo provides both the design and implementation for a complete OAuth2 provider. This provider
implements the "Authorization Code" flow as described by
[RFC 6749 section 1.3.1](https://tools.ietf.org/html/rfc6749#section-1.3.1).

Looking at the [Protocol Flow](https://tools.ietf.org/html/rfc6749#section-1.2) this repo implements
the `Authorization Server` (and your service the `Resource Server`). Included is the design and
implementation for three different types of requests:

* The request coming from the resource owner that authorizes the client. As a concrete example if
  your service implemented Facebook then the request would be the post sent by the end user upon
  accepting the "Request for Permission" displayed when adding an app such as Spotify. The result of
  this request is a redirection to a pre-configured Spotify URI with an authorization code in the
  redirect URL query string. The Spotify app can now use the authorization code to retrieve a
  refresh and access token from Facebook.

* The request coming from the client to exchange the authorization code obtained in the step above
  for a refresh and access tokens. Keeping with the Facebook example this request comes from Spotify
  after it receives the authorization code from the redirect to retrieve a refresh and access token.
  At this point Spotify can make requests on behalf of the end user using the access token to
  authenticate against our service (Facebook).

* The request coming from the client to renew an access token using a refresh token obtained in the
  step above. In the Facebook example this request would be again from Spotify when or before an
  access token expires to retrieve a fresh new access token.

The last two requests are depicted in "Figure 2: Refreshing an Expired Access Token" in
[section 1.5 of the RFC](https://tools.ietf.org/html/rfc6749#section-1.5) as steps `(A)`, `(B)` and
`(G)`, `(H)`.

The complete flow is depicted in "Figure 3: Authorization Code Flow" in
[section 4.1 of the RFC](https://tools.ietf.org/html/rfc6749#section-4.1). Remember that this repo
implements the "Authorization Server" and that the "Client" is the 3rd party service wanting to use
OAuth2 to send requests to your service on behalf of the "Resource Owner" (i.e. end user of your
service).

## Usage

Using the provider consists of two parts:

1. Import the design package in the design of the API that implements the OAuth2 provider and call
   the `OAuth2` function which creates the security scheme. Use the scheme to secure the API
   endpoints as needed. See [Design](#design) below.
2. Implement and mount the oauth2 provider controller onto the service.
   See [Implement](#implement) below.

### Design

First import the OAuth2 design package in your design:

```go
package design

import (
    . "https://github.com/Relesi/auth"
)
```

Then call the `OAuth2` function to create the OAuth2 security scheme. This function accepts three
arguments:

* `authorizationEndpoint` is the request path to the authorization endpoint as described by
https://tools.ietf.org/html/rfc6749#section-3.1. This endpoint receives requests from the
resource owner to grant access to the client.  It responds with a redirect to a preconfigured URI
and provides the authorization code as a query string.

* `tokenEndpoint` is the request path to the token endpoint as described by
https://tools.ietf.org/html/rfc6749#section-3.2. This endpoint exchanges authorization codes and
refresh tokens for access tokens.

* `dsl` is an optional anonymous function that may define scopes for the grants associated with the
access tokens.

Here is an example creating a OAuth2 security scheme using `/oauth2/authorize` and `/oauth2/token`
as endpoints and definiting two scopes `api:read` and `api:write`:

