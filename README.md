# Solid

[![](https://img.shields.io/badge/project-Solid-7C4DFF.svg?style=flat-square)](https://github.com/solid/solid)

![Solid Logo](https://avatars3.githubusercontent.com/u/14262490?v=3&s=200)

> Re-decentralizing the web

Read more about [Solid information](https://github.com/solid/information).

Solid (derived from "**so**cial **li**nked **d**ata") is a proposed set of
conventions and tools for building *decentralized Web applications* based on
[Linked Data](http://www.w3.org/DesignIssues/LinkedData.html) principles. Solid
is modular and extensible. It relies as much as possible on existing
[W3C](http://www.w3.org/) standards and protocols.

Solid is made possible by a community of [contributors](https://github.com/solid/solid/blob/master/CONTRIBUTORS.md).

## Table of Contents

1. [About Solid](#about-solid)
2. [Standards Used](#standards-used)
3. [Project directory](#project-directory)
4. [Contributing to Solid](#contributing-to-solid)
  * [Solid Project Workflow](#solid-project-workflow)

## About Solid

Specifically, Solid is:

* A tech stack -- a set of complementary
  [standards](https://github.com/solid/solid-spec#standards-used) and
  [data formats/vocabularies](https://github.com/solid/vocab) that together
  provide capabilities that are currently available only through centralized
  social media services (think Facebook/Twitter/LinkedIn/many others), such as
  *identity*, *authentication and login*, *authorization and permission lists*,
  *contact management*, *messaging and notifications*, *feed aggregation and
  subscription*, *comments and discussions*, and more.
* A **[Specifications document](https://github.com/solid/solid-spec)**
  that describes a REST API that extends those existing
  standards, contains design notes on the individual components used, and is
  intended as a guide for developers who plan to build servers or applications.
* A set of [servers](https://github.com/solid/solid-platform#servers) that
  implement this specification.
* An ecosystem of [social apps](https://github.com/solid/solid-apps),
  [identity providers](https://github.com/solid/solid-idp-list) and helper
  libraries (such as [solid-auth-client](https://github.com/solid/solid-auth-client)) that run on
  the Solid platform.
* A community providing documentation, discussion (see the
  [solid forum](https://forum.solidproject.org)), and
  [talks/presentations](https://github.com/solid/talks).

## Standards Used

The Solid platform uses the following standards.

* [RDF 1.1 (Resource Description Framework)](http://www.w3.org/RDF/)
  (see also [RDF Primer](http://www.w3.org/TR/rdf11-concepts/)) is heavily
  used in Solid data models. By default, the *preferred* RDF serialization
  format is [Turtle](http://www.w3.org/TR/turtle/). Alternative serialization
  formats such as [JSON-LD](http://www.w3.org/TR/json-ld/) and
  [RDFa](http://www.w3.org/TR/rdfa-primer/) can also be used.

* The [WebID 1.0 (Web Identity and
  Discovery)](http://www.w3.org/2005/Incubator/webid/spec/identity/)
  standard is used to provide universal usernames/IDs for Solid apps, and to
  refer to unique Agents (people, organizations, devices). See also the
  [WebID interoperability notes](http://www.w3.org/2005/Incubator/webid/wiki/Identity_Interoperability)
  for an overview of how WebID relates to other authentication and identity
  protocols.

* WebIDs, when accessed, yield
  [WebID Profile](http://www.w3.org/2005/Incubator/webid/spec/identity/#dfn-webid_profile)
  documents (in Turtle and other RDF formats).

* The [FOAF vocabulary](http://xmlns.com/foaf/0.1/) is used both in WebID
  profiles, and in specifying Access Control lists (see below).

* Authentication (for logins, page personalization and more) is done via the
  [WebID-TLS protocol](http://www.w3.org/2005/Incubator/webid/spec/tls/).
  WebID-TLS extends WebID Profiles to include references to the subject's
  [public keys](https://en.wikipedia.org/wiki/Public-key_cryptography) in
  the form of [X.509 Certificates](https://en.wikipedia.org/wiki/X.509), using
  [Cert Ontology 1.0](http://www.w3.org/ns/auth/cert) vocabulary.
  The authentication sequence is done using the
  [HTTP over TLS](https://tools.ietf.org/html/rfc2818) protocol. Unlike normal
  HTTPS use cases, WebID-TLS is done without referring to
  [Certificate Authority](https://en.wikipedia.org/wiki/Certificate_authority)
  hierarchies, and instead encourages host server-signed (or self-signed)
  certificates.

* In Solid, certificate creation is typically done in the browser using the
  HTML5 [keygen
  element](http://www.w3.org/TR/html5/forms.html#the-keygen-element),
  to provide a one-step creation and certificate publication user experience.

* Authorization and access lists are done using
  [Basic Access Control ontology](http://www.w3.org/ns/auth/acl) (see also the
  [WebAccessControl wiki page](http://www.w3.org/wiki/WebAccessControl) for
  more details).

* Solid uses the [Linked Data Platform (LDP)](http://www.w3.org/TR/ldp/)
  standard (see also [LDP Primer](http://www.w3.org/TR/ldp-primer/))
  extensively, as a standard way of reading and writing generic Linked Data
  resources.

## Solid Platform Notes

Solid applications are somewhat like multi-user applications where instances talk
to each other through a shared filesystem, and the Web is that filesystem.

1. The [LDP specification](http://www.w3.org/TR/ldp/) defines a set of rules for
  HTTP operations on Web resources, some based on [RDF](http://www.w3.org/RDF/),
  to provide an architecture for reading and writing Linked Data on the Web. The
  most important feature of LDP is that it provides us with a standard way of
  RESTfully writing resources (documents) on the Web, without having to rely on
  less flexible conventions (APIs) based around sending form-encoded data using
  POST. For more insight into LDP, take a look at the examples in the LDP
  [Primer document](http://www.w3.org/TR/ldp-primer/).

2. Solid's basic protocol is REST, as refined by LDP with minor extensions. New
  items are created in a *container* (which could be called a collection or
  directory) by sending them to the container URL with an HTTP POST or issuing
  an HTTP PUT within its URL space. Items are updated with HTTP PUT or HTTP
  PATCH. Items are removed with HTTP DELETE. Items are found using HTTP GET and
  following links. A GET on the container returns an enumeration of the items in
  the container.

3. Servers are application-agnostic, so that new applications can be developed
  without needing to modify servers. For example, even though the [LDP
  1.0](http://www.w3.org/TR/ldp/) specs contains nothing specific to
  "social", many of the [W3C Social Work Group](http://www.w3.org/Social/WG)'s
  [User Stories](http://www.w3.org/wiki/Socialwg/Social_syntax/User_Stories) can
  be implemented using only **application logic**, with no need to change code on
  the server. The design ideal is to keep a small standard data management core
  and extend it as necessary to support increasingly powerful classes of
  applications.

4. The data model is RDF. This means the data can be transmitted in various
  syntaxes like [Turtle](http://www.w3.org/TR/turtle/),
  [JSON-LD](http://www.w3.org/TR/json-ld/) (JSON with a "context"), or
  [RDFa](http://www.w3.org/TR/rdfa-primer/) (HTML attributes). RDF is
  REST-friendly, using URLs everywhere, and it provides **decentralized
  extensibility**, so that a set of applications can cooperate in sharing a new
  kind of data without needing approval from any central authority.

## Project directory

- Useful links
  - [Specification](https://github.com/solid/solid-spec)
  - [Talks](https://github.com/solid/talks)
  - Tutorials
    - [Intro to Solid Tutorial](https://github.com/solid/solid-tutorial-intro)
      introductory Solid tutorial using the
      [solid-auth-client](https://github.com/solid/solid-auth-client) library
    - [Solid Angular Apps Tutorial](https://github.com/solid/solid-tutorial-angular):
      a set of example apps built on the Solid platform using AngularJS.
    - [Working with RDF data](https://github.com/solid/solid-tutorial-rdflib.js) tutorial about using a library called [rdflib.js](https://github.com/linkeddata/rdflib.js)

- Implementing
  - [Sign-up/Login application](https://github.com/solid/solid-signup)
  - Solid Server Test suite

- Community
  - [Server implementations](https://github.com/solid/solid-platform)
  - [Identity providers](https://github.com/solid/solid-idps)
  - [Applications](https://github.com/solid/solid-apps)
  - Join our conversations in our [gitter chat room](https://gitter.im/solid/chat)


## Contributing to Solid

### Get a WebID

In order to try out some of the apps built using Solid, you will need typically an identity on some solid server.
There are two forms of authentication we use, and so two types of account.

#### WebID-OIDC

This uses OpenID Connect to give you a WebID. It involves signing in with a password at your chosen
identity provider, such as (2018/2) [solid.community](https://solid.community/), or [solidtest.space](https://solidtest.space/). 

#### WebID_TLS

A WebID profile from one of the Solid-compliant [identity providers](https://solid.github.io/solid-idps/), such as [databox.me](https://databox.me/),  

With WebID-TLS, you will need to make a WebID browser certificate from the above profile (this is usually created
when you sign up for a WebID profile account, but it only works on Firefox at the moment (2018)).

### Running a server

Additionally, to get started with developing for the Solid platform, you'll
need:

1. A Solid-compliant [server](https://github.com/solid/solid-platform#servers).

2. While not required, an understanding of
  RDF/[Turtle](http://www.w3.org/TR/turtle/) principles and
  [Linked Data Platform](http://www.w3.org/TR/ldp-primer/) concepts will help
  you understand the general workflow.

### Solid Project Workflow

To contribute to Solid development, and to bring up issues or feature requests,
please use the following workflow:

1. Have a question or a feature request or a concern about the Solid framework,
  or on one of its servers? **Open an issue on
  [solid/solid](https://github.com/solid/solid)** (this repo here).

2. Have an issue with the *Solid spec* specifically? **Open an issue on
  [solid/solid](https://github.com/solid/solid) anyway.** And then, as a result of
  discussion, if it's agreed that it is actually a Spec issue, it will be moved
  to `solid-spec`.

3. The individual [solid/solid](https://github.com/solid) issues can coordinate
  and track component/dependent issues on the various affected Solid servers,
  apps, and so on.
  
## Places to chat
https://github.com/solid/information#solid-conversations
 
 
