This package provides a base for integrating with [OAuth 2.0](http://oauth.net/2/) service providers.

[![Gitter Chat](https://img.shields.io/badge/gitter-join_chat-brightgreen.svg?style=flat-square)](https://gitter.im/thephpleague/oauth2-client)
[![Source Code](https://img.shields.io/badge/source-thephpleague/oauth2--client-blue.svg?style=flat-square)](https://github.com/thephpleague/oauth2-client)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](https://github.com/thephpleague/oauth2-client/blob/master/LICENSE)
[![Build Status](https://img.shields.io/github/workflow/status/thephpleague/oauth2-client/CI?label=CI&logo=github&style=flat-square)](https://github.com/thephpleague/oauth2-client/actions?query=workflow%3ACI)
[![Codecov Code Coverage](https://img.shields.io/codecov/c/gh/thephpleague/oauth2-client?label=codecov&logo=codecov&style=flat-square)](https://codecov.io/gh/thephpleague/oauth2-client)
[![Total Downloads](https://img.shields.io/packagist/dt/league/oauth2-client.svg?style=flat-square)](https://packagist.org/packages/league/oauth2-client)

---

The OAuth 2.0 login flow, seen commonly around the web in the form of "Connect with Facebook/Google/etc." buttons, is a common integration added to web applications, but it can be tricky and tedious to do right. To help, we've created the `league/oauth2-client` package, which provides a base for integrating with various OAuth 2.0 providers, without overburdening your application with the concerns of [RFC 6749](http://tools.ietf.org/html/rfc6749).

This OAuth 2.0 client library will work with any OAuth 2.0 provider that conforms to the OAuth 2.0 Authorization Framework. Out-of-the-box, we provide a `GenericProvider` class to connect to any service provider that uses [Bearer tokens](http://tools.ietf.org/html/rfc6750). See our [basic usage guide](https://oauth2-client.thephpleague.com/usage/) for examples using `GenericProvider`.

Many service providers provide additional functionality above and beyond the OAuth 2.0 specification. For this reason, you may extend and wrap this library to support additional behavior. There are already many [official](https://oauth2-client.thephpleague.com/providers/league/) and [third-party](https://oauth2-client.thephpleague.com/providers/thirdparty/) provider clients available (e.g., Facebook, GitHub, Google, Instagram, LinkedIn, etc.). If your provider isn't in the list, feel free to add it.

This package is compliant with [PSR-1][], [PSR-2][], [PSR-4][], and [PSR-7][]. If you notice compliance oversights, please send a patch via pull request. If you're interested in contributing to this library, please take a look at our [contributing guidelines](https://github.com/thephpleague/oauth2-client/blob/master/CONTRIBUTING.md).

To access YouTrack resources with the REST API using OAuth 2.0, you need to log in, authorize, and obtain an access token for YouTrack. Authorization is provided by the Hub service.

We only recommend that you use OAuth 2.0 when your application requires client-side authentication. Otherwise, you're better off using token-based authorization. For more information, see Permanent Token Authorization.

			Do not attempt to register external services in the built-in Hub service! This can cause the service to crash or malfunction. It can also result in a corrupted database or data loss.

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
   

### Design

First import the OAuth2 design package in your design:

```go
package br.com.relesi.cloud.services.auth;

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

