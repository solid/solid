# Decentralized Authentication Glossary

In order to discuss decentralized authentication protocol details, it would be
helpful to familiarize ourselves with the terminology that is frequently used
by various decentralized protocol specs (such as OAuth2, OpenID Connect).

##### User
Human user. If the user is an app or service (that has its own WebID Profile),
this can be generalized to `Agent`. Also called `Resource Owner`. In the
following examples, Alice and Bob are Users.

##### User-Agent
A formal name for a `Browser`. Note that this is often separate from a Client
application (in many cases, client apps written in Javascript run *inside* the
browser).

##### Identity Provider (OP)
An OpenID Connect Identity `Provider` (called `OP` in most OIDC specs). Also
sometimes referred to as `Issuer`. This can be either a POD (see below) or an
external OIDC provider such as
[Google](https://developers.google.com/identity/protocols/OpenIDConnect). In
the spec, Alice's POD, `alice.com`, will mostly play the role of a Provider.

##### Resource Server
A server hosting resources that the user wants to access, such as HTML, images,
Linked Data / RDF sources, and so on. In the spec, `bob.com` will be used as
the `Resource Server` (that is, Alice will be requesting resources on Bob's
server). *Note:* In the traditional federated social sign on context, a
provider (such as Facebook) serves as *both* an Identity Provider *and* a
Resource Server.

##### Relying Party (RP)
A `Relying Party` is a POD or a client app that has to *rely* on an ID Token
that's issued by a Provider. In the spec, when Alice tries to access a resource
on `bob.com`, Bob's POD acts as the Relying Party, in that interaction.
And correspondingly, Alice's POD, `alice.com`, will serve as the Identity
Provider, again for that interaction.

Incidentally, when Alice tries to access a resource on her *own* POD,
`alice.com` plays all of the roles -- it's both the Provider and a Relying
Party (as well as the Resource Server).

##### POD
A Personal Online Datastore (POD for short). It plays several roles -- firstly,
it stores a user's data (and so acts as a Resource Server). In many cases, it
also hosts the user's WebID Profile, and implements the API endpoints that allow
it to act as a WebID-OIDC Identity Provider (OP). Lastly, when users requests
resources from it, the POD also acts as a Relying Party (a recipient of those
users' ID Tokens).
In this spec, `alice.com` and `bob.com` are both PODs.

##### Home POD vs Other POD
A user's Home POD is one that hosts their WebID Profile, and also acts as that
user's Identity Provider. We use the term *Other POD* in this spec to denote
some other WebID-OIDC compliant POD, acting as a Resource Server and Relying
Party, that a user is trying to access using the WebID URI and Profile of their
Home POD.

When Alice tries to access a resource on Bob's POD, `alice.com` is her Home
POD, and `bob.com` plays the role of the Other POD.

##### Public Client vs Confidential Client
Public - in-browser, mobile or desktop app, can't be trusted with secrets.
Confidential - server-side app, can be trusted with secrets.

##### Presenter
A public client app that is trying to *present* a user's credentials from their
home POD to some other POD. For example, Bob is trying to access, via a client
app, a shared file on Alice's `alice.com` POD, logging in using his own
`bob.com` POD/provider. In this example, `bob.com` is a Provider, `alice.com` is
a Relying Party, and the client app (say, a browser-based HTML editor) is a
Presenter.
