# WebID-OIDC spec

## Table of Contents

* [Introduction](#introduction)
* [Motivation](#motivation)
  - [Benefits and Capabilities](#benefits-and-capabilities)
  - [If You're Familiar with WebID](#if-youre-familiar-with-webid)
  - [If You're Familiar with OAuth 2 or OpenID Connect](#if-youre-familiar-with-oauth-2-or-openid-connect)
* [Brief Workflow Summary](#brief-workflow-summary)
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

 * Read the [Brief Workflow Summary](#brief-workflow-summary) section below
 * Refer to the [Decentralized Authentication
   Glossary](decentralized-auth-glossary.md) to help clarify how the various
   terms (Relying Party, Provider, etc) apply to WebID systems.
 * Read the [OpenID Connect
   explained](http://connect2id.com/learn/openid-connect) article. Becoming
   familiar with the basic OIDC concepts will be quite helpful with
   understanding this spec.

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

To walk through a more detailed example for WebID-OIDC sign in, refer to the
[Example WebID-OIDC Workflow](example-workflow.md) doc.
