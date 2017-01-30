# WebID-OIDC spec

## Table of Contents

* [Introduction](#introduction)
* [Motivation](#motivation)
  - [Benefits and Capabilities](#benefits-and-capabilities)
  - [If You're Unfamiliar with OIDC](#if-youre-unfamiliar-with-oidc)
* [Differences from Classic OpenID Connect](#differences-from-classic-openid-connect)
* [Brief Workflow Summary](#brief-workflow-summary)
* [Enhancement: Add `webid` Claim to ID Token](#enhancement-add-webid-claim-to-id-token)
* [Enhancement: WebID Provider Confirmation](#enhancement-webid-provider-confirmation)
* [Detailed Sign In Workflow Example](#detailed-sign-in-workflow-example)

## Introduction

WebID-OIDC is an authentication delegation protocol (as well as a toolkit of
useful auth-related verification techniques) suitable for WebID-based
decentralized systems such as [Solid](https://github.com/solid/solid), as well
as most LDP-based systems.

The end result of any WebID authentication workflow is a verified WebID
URI (specifically, the recipient verifies that the agent controls that URI).
For example,
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

## If You're Unfamiliar with OIDC

If you're not familiar with the OIDC (or OAuth 2) workflow, you should do the
following:

 * Read the [Brief Workflow Summary](#brief-workflow-summary) section below
 * Refer to the [Decentralized Authentication
   Glossary](decentralized-auth-glossary.md) to help clarify how the various
   terms (Relying Party, Provider, etc) apply to WebID systems.
 * Read the [OpenID Connect explained](http://connect2id.com/learn/openid-connect)
   article. Becoming familiar with the basic OIDC concepts will be quite
   helpful with understanding this spec.

## Differences from Classic OpenID Connect

WebID-OIDC makes the following enhancements to the base OpenID Connect protocol
(which itself improves and builds on OAuth 2):

* Discusses and formalizes the [Provider Selection](example-workflow.md#21-provider-selection)
  step.
* Adds an [new `webid` claim](#enhancement-add-webid-claim-to-id-token) to the
  ID Token format. This will contain
  the WebID URI, a globally unique user id (which also happens to yield a
  useful user profile when dereferenced). This claim, instead of the
  traditional `sub`ject claim, will be used by the recipient to uniquely
  identify the user. (The spec also has fallback provisions for situations
  where adding a claim to the ID Token is impossible.)
* Adds an additional step: [WebID Provider Confirmation](#enhancement-webid-provider-confirmation).
  After the WebID URI is extracted, the recipient of the ID Token must confirm
  that the Provider was indeed authorized by the holder of the WebID profile.

## Brief Workflow Summary

The overall sign in workflow used by the WebID-OIDC protocol is as follows.
For example, here is what happens when Alice tries to request the resource
`https://bob.com/resource1`.

1. [Initial Request](example-workflow.md#1-initial-request): Alice
   (unauthenticated) makes a request to `bob.com`, receives an HTTP `401
   Unauthorized` response, and is presented with a 'Sign In With...' screen.
2. [Provider Selection](example-workflow.md#21-provider-selection): She selects
   her WebID service provider by clicking on a logo, typing in a URI (for
   example, `alice.databox.me`), or entering her email.
3. [Local
   Authentication](example-workflow.md#3-local-authentication-to-provider):
   Alice gets redirected towards her service provider's own Sign In page,
   `https://alice.databox.me/signin`, and authenticates using her preferred
   method (password, WebID-TLS certificate, FIDO 2 /
   [WebAuthn](https://w3c.github.io/webauthn/) device, etc).
4. [User Consent](example-workflow.md#4-user-consent): (Optional) She'd also be
   presented with a user consent screen, along the lines of "Do you wish to
   sign in to `bob.com`?".
5. [Authentication Response](example-workflow.md#5-authentication-response):
   She then gets redirected back towards `https://bob.com/resource1` (the
   resource she was originally trying to request). The server, `bob.com`, also
   receives a signed ID Token from `alice.databox.me`, attesting that she has
   signed in.
6. [Deriving a WebID URI](#enhancement-add-webid-claim-to-id-token):
   `bob.com` (the server controlling the resource) validates the ID Token, and
   extracts Alice's WebID URI (see the `webid` claim section below) from
   inside  it. She is now signed in to `bob.com` as user
   `https://alice.databox.me/#i`.
7. [WebID Provider Confirmation](#enhancement-webid-provider-confirmation):
   `bob.com` makes an HTTP request to Alice's WebID URI,
   `https://alice.databox.me/` and *confirms* (via parsing the `oidc` link
   header relation) that Databox is indeed Alice's Provider of choice.

There is a lot of heavy lifting happening under the hood, performed by
`bob.com` and `alice.databox.me`, the two servers involved in this exchange.
They establish a trust relationship with each other (via
[Discovery](example-workflow.md#22-provider-discovery), and
[Dynamic Registration](example-workflow.md#23-dynamic-client-registration-first-time-only)), they
verify each other's signatures against their public keys, and verify Alice's
client app (if she's using one). Fortunately, all of that complexity is hidden
from the user (and most of it is also hidden from the app developer).

## Enhancement: Add `webid` Claim to ID Token

WebID-OIDC uses the following algorithm to obtain a WebID URI from an ID Token.

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

## Enhancement: WebID Provider Confirmation

#### The Problem

The OIDC spec uses the ID Token's `sub`ject claim as a unique user id. However,
it requires that the id is unique *for a given Provider*. Given that a WebID
is a *globally* unique user identifier, the WebID-OIDC protocol needs to take
an additional step and *confirm* that the holder of that WebID has authorized
a given Provider to use that WebID. Otherwise, the following situation can
happen:

1. Alice logs in to `bob.com` with her identity Provider of choice, `alice.com`.
   The ID Token from `alice.com` contains alice's WebID is
   `https://alice.com/#i`. So far so good.
2. An attacker also logs in to `bob.com`, using `evilbox.com` as an identity
   Provider. And because they happen to control that server, they can put
   anything they want in the `webid` claim of any ID Token coming out of that
   server. So they *also* claim that their `webid` is `https://alice.com/#i`.

Without an additional confirmation step, how can a recipient of an ID Token
(here, `bob.com`) know which of those login attempts is correct? To put it
another way, how can a recipient know which Provider is *authorized* by the
owner of the WebID?

#### The Solution

At the end of the WebID-OIDC workflow, a recipient (Relying Party) of the ID
Token MUST confirm that the Identity Provider (the value in the `iss`uer claim)
is authorized by the holder of the WebID, by doing the following:

1. Dereference (make an HTTP GET request) to the WebID URI.
2. Find the preferred/authorized *provider URI* for that WebID URI by:
   * First, check the `Link:` header for the `oidc` link relation. If present,
     use the value of that link relation as the authorized Provider URI.
   * If the `oidc` link relation header is not present, attempt to parse the
     body of the WebID Profile, and parse the preferred/authorized Provider URI
     from it. (TODO: determine which RDF predicate to use.)
3. If the Provider URI is not present (either in the header or the body of the
   WebID Profile, the Relying Party MUST reject the sign in attempt.
4. If the Provider URI is present, it MUST match the Provider URI in the ID
   Token (the `iss` claim), and reject the sign in attempt if not matching.

## Detailed Sign In Workflow Example

To walk through a more detailed example for WebID-OIDC sign in, refer to the
[Example WebID-OIDC Workflow](example-workflow.md) doc.
