# WebID-OIDC spec

## Table of Contents

* [Introduction](#introduction)
* [Motivation](#motivation)
  - [Benefits and Capabilities](#benefits-and-capabilities)
  - [If You're Familiar with WebID](#if-youre-familiar-with-webid)
  - [If You're Familiar with OAuth 2 or OpenID Connect](#if-youre-familiar-with-oauth-2-or-openid-connect)
* [Brief Workflow Summary](#brief-workflow-summary)
* [Decentralized Authentication Terminology](#decentralized-authentication-terminology)
* [Detailed Sign In Workflow](#detailed-sign-in-workflow)

## Introduction

WebID-OIDC is an authentication delegation protocol (as well as a toolkit of
useful auth-related verification techniques) suitable for WebID-based
decentralized systems such as [Solid](https://github.com/solid/solid), as well
as most LDP-based systems.

The end result of any WebID authentication workflow is a verified WebID
URI. For example,
[WebID-TLS](https://github.com/solid/solid-spec/blob/master/authn-webid-tls.md)
derives the WebID URI from a TLS certificate, and verifies the certificate
against the public key in an agent's WebID Profile. Similarly, the end result of
[OpenID Connect (OIDC)](https://openid.net/specs/openid-connect-core-1_0.html)
workflows is a verified ID Token. The WebID-OIDC protocol specifies a mechanism
for getting a WebID URI from an OIDC ID Token, and gains the benefits of both
the decentralized flexibility of WebID, and the field-proven security of OpenID
Connect.

## Motivation

The Solid team, while [researching additional authentication
mechanisms](https://github.com/solid/solid/issues/22) to run alongside the
existing one (WebID-TLS), performed a review of existing authentication
protocols that has been used by (or proposed for) decentralized application
ecosystems. Several criteria were kept in mind.

One, we wanted to avoid the common pitfalls that are often encountered
when dealing with authentication for decentralized systems: the use of HTTP
Basic or Digest Auth, and relying on Bearer Tokens as an authentication
mechanism.

Secondly, we were looking for a protocol that did not rely solely on identifying
users via public/private crypto keys. Aside from the general difficulty people
have with storing and managing crypto keys, public client applications (browser
side Javascript apps, for example) do not have access to private key storage
facilities or keychain APIs. (And even the upcoming WebAuthn spec requires the
user to *register* an account for each domain -- precisely the situation that
most decentralized social app projects like Solid are trying to fix.) This
ruled out proposals such as HTTP Signatures, most "Identity on the Blockchain"
proposals including Bitcoin, and other client-side key management based systems.

If at all possible, we were looking for protocols that used standards and tech
stacks that were familiar and comprehensible to app developers (which ruled out
the XML-based and enterprise-heavy SAML, as well as earlier incarnations of
OpenID).

Other properties that we were looking for in a protocol included:

* Support for the full range of app and agent types, including browser side
  Javascript apps, traditional server-side web apps, mobile apps and desktop
  apps, and so on.
* Was well-spec'd and widely deployed
* Addressed and incorporated security best practices and recommendations (see,
  for example [RFC 4962](https://tools.ietf.org/html/rfc4962) and
  [RFC 6819](http://tools.ietf.org/html/rfc6819)), such as fresh strong session
  keys, cryptographic algorithm independence, authentication of all parties
  involved (user, client app, server, etc), and proof against common attacks
  (replay attacks, man-in-the-middle, token hijacking and many others).
* Provided a familiar end-user experience
* Was usable on public or shared computers (had reasonable sign-out
  capabilities, etc)
* Was compatible with the other complementary Solid specs (such as the Web
  Access Control authorization system)
* Had support for down-the-road capability requirements, such as access
  revocation, blacklists and whitelists for both apps and service providers,
  the possibility of confidentiality, pre-authorized "unattended" access, and
  so on.

### Why OpenID Connect

We have found OpenID Connect the only protocol that fit all of those
requirements (or even came close). At its heart, OIDC is a general purpose
toolkit for authentication and delegation, with provisions and specifications
for:

- User authentication via methods both familiar (such as username and password,
  WebID-TLS browser side certificates) and upcoming (such as the FIDO 2/
  WebAuthn standard)
- Verification of identity providers
- Verification of client applications of all types (browser side,
  server side, desktop and mobile, etc)
- Session expiration

### Benefits and Capabilities

* Fully decentralized cross-domain authentication
* Builds on decades of real-world authentication industry experience.
* Incorporates lessons from, and fixes to threat models of: SAML, OpenID and
  OpenID 2, OAuth and OAuth 2. See, for example,
  [RFC 6819 - OAuth 2.0 Threat Model and Security
  Considerations](http://tools.ietf.org/html/rfc6819) -- OpenID Connect was
  developed in large part to address the threats outlined there.
* Stands on the shoulders of giants (makes use of the JOSE suite of standards
  for token representation, cryptographic signing and encryption,
  including [JWT](https://tools.ietf.org/html/rfc7519),
  [JWA](https://tools.ietf.org/html/rfc7518),
  [JWE](https://tools.ietf.org/html/rfc7516) and
  [JWS](https://tools.ietf.org/html/rfc7515))
* Sign Off (and Single Sign Off) capability
* Capability for revocations, black lists and white lists
* Supports authentication for the full range of agents and clients: in-browser
  Javascript apps, traditional server-side web apps, mobile and desktop apps,
  and IoT devices.
* Drop-in compatibility with existing Web Access Control ACL implementations
  such as those in Solid servers.
* Sets up the infrastructure for adding Capabilities functionality to Solid

### If You're Familiar with WebID

WebID-OIDC uses the concept of a WebID URI to enhance the existing OpenID
Connect protocol.

If you're not familiar with the OIDC (or OAuth 2) workflow, you should do the
following:

1. Read the [Brief Workflow Summary](#brief-workflow-summary) section below
2. Read the [OpenID Connect
   explained](http://connect2id.com/learn/openid-connect) article. Becoming
   familiar with the basic OIDC concepts will be quite helpful with
   understanding this spec.
3. Read the rest of the spec

### If You're Familiar with OAuth 2 or OpenID Connect

WebID-OIDC makes the following enhancements to the base OpenID Connect protocol
(which itself improves and builds on OAuth 2):

* Discusses, formalizes and specifies the [Provider
  Selection](#21-provider-selection) step.
* Adds an additional claim to the ID Token format: `webid`. This will contain
  the WebID URI, a globally unique user id (which also happens to yield a
  useful user profile when dereferenced). This claim, instead of the
  traditional `sub`ject claim, will be used by the recipient to uniquely
  identify the user. (The spec also has fallback provisions for situations
  where adding a claim to the ID Token is impossible.)

## Brief Workflow Summary

The overall sign in workflow used by the WebID-OIDC protocol is as follows.
For example, here is what happens when Alice tries to request the resource
`https://bob.com/resource1`.

1. [Initial Request](#1-initial-request): Alice (unauthenticated) makes a
   request to `bob.com`, receives an HTTP `401 Unauthorized` response, and is
   presented with a 'Sign In With...' screen.
2. [Provider Selection](#21-provider-selection): She selects her WebID service
   provider by clicking on a logo, typing in a URI (for example,
   `alice.databox.me`), or entering her email.
3. [Local Authentication](#3-local-authentication-to-provider): Alice gets
   redirected towards her service provider's own Sign In page,
   `https://alice.databox.me/signin`, and authenticates using her preferred
   method (password, WebID-TLS certificate, FIDO 2 /
   [WebAuthn](https://w3c.github.io/webauthn/) device, etc).
4. [User Consent](#4-user-consent): (Optional) She'd also be presented with a
   user consent screen, along the lines of "Do you wish to sign in to
   `bob.com`?".
5. [Authentication Response](#5-authentication-response): She then gets
   redirected back towards `https://bob.com/resource1` (the resource she was
   originally trying to request). The server, `bob.com`, also receives a signed
   ID Token from `alice.databox.me`, attesting that she has signed in.
6. [Extract WebID](#6-deriving-a-webid-uri-from-an-oidc-id-token): `bob.com`
   (the server controlling the resource) validates the ID Token, and extracts
   Alice's WebID URI from inside it. She is now signed in to `bob.com` as user
   `https://alice.databox.me/#i`.

There is a lot of heavy lifting happening under the hood, performed by
`bob.com` and `alice.databox.me`, the two servers involved in this exchange.
They establish a trust relationship with each other (via
[Discovery](#22-provider-discovery), and
[Dynamic Registration](#23-dynamic-client-registration-first-time-only)), they
verify each other's signatures against their public keys, and verify Alice's
client app (if she's using one). Fortunately, all of that complexity is hidden
from the user (and most of it is also hidden from the app developer).

## Detailed Sign In Workflow

Note: For the purposes of this spec, we're going to leave out the question of
Authorization (access control), and assume that the POD is performing
appropriate access control checks, and that the user has access to the resources
they are requesting.

### 1. Initial Request

#### 1a. Direct Browser Request

*Example 1:* Alice types in `https://bob.com/images/image1.png` into her
browser's address bar (or follows a link to it).

*Example 1:* `bob.com` responds with a `401 Unauthorized` response, with HTML in
the body that says something like "Welcome to Bob.com. Please enter your email
or WebID URL", and provides a text field.

A Browser makes a request to a POD acting as a Resource Server and
Relying Party. This is a direct browser request, such as when a user follows a
link or types in a URL in the address bar. No existing cookie is present, for
either the Provider POD (`alice.com`) or the Relying Party POD (`bob.com`).

The Relying Party, (here, `bob.com`) responds with an HTTP `401 Unauthorized`.
The *body* of the 401 response SHOULD contain human-readable HTML, containing
either a Select Provider form, or a meta-refresh redirect to a Select Provider
page. This HTML response starts the user on the Sign In workflow.

#### 1b. AJAX or API Client Request
A REST client (either for a server-side app, or an in-browser AJAX client)
makes an HTTP request to a POD acting as a Resource Server and Relying Party.
No existing cookie or session is present, for either the Provider POD or the
Relying Party POD.

The server (RP POD) responds with an HTTP `401 Unauthorized`. If this is an
interactive client, it is the client's responsibility to initiate the Sign In
workflow by presenting the user with a Select Provider UI.

### 2. Provider Selection and Discovery

#### 2.1 Provider Selection
*Example 1:* Alice (still on the `401 Unauthorized` page of the Relying Party
`bob.com`) enters the URI of her Home POD, `alice.com`, during the Provider
Selection step.

If the user's Provider preference is not saved (in a cookie or local storage),
at this point they must be prompted with the Provider Selection UI. The user
must indicate which POD or identity Provider they wish to use, for signing in.
Any and all of the following methods may be used:

 * User enters in the domain of their identity Provider (such as `databox.me`).
   (If no protocol is specified by the user, `https` should be assumed.)
 * User clicks on a provider's logo
 * User enters in their full WebID URI
 * The user enters in their email. The RP then uses the WebFinger protocol (with
   a fallback to WebFist), as recommended by the OIDC spec, to discover the URI
   of the Provider.

#### 2.2 Provider Discovery
*Example 1:* The Relying Party, `bob.com`, performs OIDC provider metadata
discovery on `https://alice.com`, and loads its public keys, API endpoints,
and other metadata.

If the URI of the user's preferred identity Provider was entered during Provider
Selection, the Relying Party must perform Provider Discovery to obtain
[Provider Metadata](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata)
This metadata includes useful information such as:
 - API endpoints (for authorization, client registration, and so on)
 - The Provider's public keys, which can be used to verify signed tokens
 - Crypto algorithms supported
 - Links to Policy and Terms of Service documents

See the section [Obtaining OpenID Provider Configuration
Information](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)
of the OIDC Discovery spec, for more information.

#### 2.3 Dynamic Client Registration (First Time Only)
*Example 1:* Since this is the first time the Provider (`alice.com`) and the
Relying Party (`bob.com`) have interacted, `bob.com` must dynamically register
itself *as a client/relying party* to `alice.com`. Bob's POD (in the RP role)
performs dynamic registration with Alice's POD (in the Provider role), and as
a result `bob.com` receives its very own `client_id` which identifies it
uniquely as a Relying Party to `alice.com`.

If this is the first time a Provider and a Relying Party are encountering each
other, the RP must perform
[Dynamic Client Registration](https://openid.net/specs/openid-connect-registration-1_0.html).
Note: This is an operation that happens under the hood, and does not involve the
user. All compliant OIDC clients have this functionality built in.

### 3. Local Authentication to Provider
*Example 1:* After performing Provider Selection and Discovery, `bob.com`
redirects Alice's browser (via a `302 Found` redirect) to her own Provider's
sign in page, `https://alice.com/signin`. Alice signs into `alice.com` using
a username and password.

The user is redirected to their preferred Provider's Sign In page.

The exact mechanism used to sign in is up to the provider, and all of the usual
choices apply (WebID-TLS browser-side certificates, username and password,
hardware-based FIDO 2/WebAuthentication devices, federated sign-in with the
likes of Github/Facebook/Google and so on).

If the user does not have an account with the provider, the Sign Up/Account
Creation step would happen at this point, instead.

(**Optimization**) The Provider may choose to establish a user session (via a
browser cookie), so that the user can skip Step 3 in subsequent interactions
(until the session expires or the user signs out).

### 4. User Consent
User consent screen ("Do you want to sign in to www.example.com?" etc).

*Example 1:* `alice.com`, in the Provider role, presents Alice with a User
Consent screen that says something like "Do you want to sign in to bob.com?",
and she clicks OK.

### 5. Authentication Response
After the user has signed in and provided consent, the Provider performs client
verification.

*Example 1:* `alice.com`, in the Provider role, verifies the Relying Party
(`bob.com`) that initiated the authentication process, and redirects Alice's
browser to a pre-registered `redirect_uri` that `bob.com` provided during
the previous Dynamic Registration step. So, Alice's browser gets redirected
to `https://bob.com/auth-response` (which is the URI Bob's POD uses as an OIDC
`redirect_uri` endpoint).

### 6. Deriving a WebID URI from an OIDC ID Token
*Example 1:* At this point, Alice is back to the original resource she was
trying to request. As part of the Authentication Response from `alice.com`,
the Relying Party `bob.com` has received an ID token that contains, among
other things, a `webid` property. `bob.com` validates the ID Token (makes
sure it has not expired, that the signature matches `alice.com`'s public key,
and so on). The contents of the ID Token's `webid` claim is Alice's WebID URI,
`https://alice.com/#i`.

**Step 1:** Look in the ID Token for the `webid` claim. This claim is added to
the set of [OpenID Connect ID
Token](https://openid.net/specs/openid-connect-core-1_0.html#IDToken) claims by
this WebID-OIDC spec. (Note that the set of ID Token claims is extensible, by
design, as explained in the OIDC Core spec.) If the `webid` claim is present in
the ID Token, its contents should be used as the WebID URI by the Relying Party,
instead of the traditional `sub` (subject identifier) claim.

**Step 2 (Optional):** If the `webid` claim is not present in the ID Token, the
Relying Party should proceed to make an OIDC [UserInfo
Request](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo), with
the appropriate Access Token that it received alongside the ID Token. This
fallback procedure is provided for cases where users do not have control over
the contents of the ID Tokens issued by their Providers and so would not be able
to use the `webid` extension claim. This would be the case, for example, if a
user wanted to sign in to a WebID-OIDC Relying Party using an existing
mainstream Provider such as Google. Once the UserInfo response is received by
the Relying Party, the standard `website` claim should be used as the WebID URI
by that RP.

(**Optimization**) The Relying Party may choose to establish a user session
(via a browser cookie), so that the user can skip Steps 1-5 in subsequent
interactions (until the session expires or the user signs out).
