<div align="center">
<a href="https://solidproject.org/">
    <img src="https://avatars3.githubusercontent.com/u/14262490?v=3&s=200" alt="Solid logo" title="Solid" align="center" height="60" />
</a>

<h3>Solid</h3>

[![](https://img.shields.io/badge/project-Solid-7C4DFF.svg?style=flat-square)](https://github.com/solid/solid)

Welcome to [The Web — Take 3](https://solidproject.org/take3). This time, it's personal.

Read more at [Solid's Official Website »](https://solidproject.org/)

</div>

---

<h2>Table of Contents</h2>

- [What is Solid](#what-is-solid)
- [How to Get Started](#how-to-get-started)
- [Standards](#standards)
- [How Solid Works](#how-solid-works)
- [How to Contribute to this Repo](#how-to-contribute-to-this-repo)
- [How to Join Our Community](#how-to-join-our-community)
    - [1. Solid Forum](#1-solid-forum)
    - [2. Solid Chat](#2-solid-chat)
- [How to Learn More](#how-to-learn-more)
- [Contributors](#contributors)

---


## What is Solid

Solid is a technology, like the Web, but a new level of standard which adds to the existing protocols to make it more powerful, particularly to empower individuals at home and at work.

Now let's break this down.

Solid is...

* A tech stack — a set of complementary [standards](https://github.com/solid/solid-spec#standards-used) and [data formats/vocabularies](https://github.com/solid/vocab) that together provide capabilities that are currently available only through centralized social media services (think Facebook/Twitter/LinkedIn/many others), such as *identity*, *authentication and login*, *authorization and permission lists*, *contact management*, *messaging and notifications*, *feed aggregation and subscription*, *comments and discussions*, and more.
* A [Specifications document](https://github.com/solid/solid-spec) that describes a REST API that extends those existing standards, contains design notes on the individual components used, and is intended as a guide for developers who plan to build servers or applications.
* A set of [Pod Servers](https://solidproject.org/for-developers/pod-server) that implement this specification.
* An ecosystem of [Solid Apps](https://solidproject.org/use-solid/apps), [Providers](https://solidproject.org/use-solid/) and helper libraries (such as [solid-auth-client](https://github.com/solid/solid-auth-client)) that run on the Solid platform.
* A [community](#how-to-join-our-community) providing documentation and discussion.

## How to Get Started

1. Start using Solid by [getting a Pod and a WebID](https://solidproject.org/use-solid/).

2. Begin using [Solid Apps](https://solidproject.org/use-solid/apps).

3. If you're a developer, [try building a Solid app](https://solidproject.org/for-developers/apps).

4. If you're technically inclined, [set up your own Pod Server](https://solidproject.org/for-developers/pod-server).

5. If you're part of an Enterprise, [let us help you enter the web's next phase](https://solidproject.org/for-enterprises/).

6. You can also help by [contributing to this repository](#how-to-contribute-to-this-repo).

## Standards

The Solid platform uses the following standards.

* [RDF 1.1 (Resource Description Framework)](http://www.w3.org/RDF/) (see also [RDF Primer](http://www.w3.org/TR/rdf11-concepts/)) is heavily used in Solid data models. By default, the *preferred* RDF serialization format is [Turtle](http://www.w3.org/TR/turtle/). Alternative serialization formats such as [JSON-LD](http://www.w3.org/TR/json-ld/) and [RDFa](http://www.w3.org/TR/rdfa-primer/) can also be used.

* The [WebID 1.0 (Web Identity and Discovery)](http://www.w3.org/2005/Incubator/webid/spec/identity/) standard is used to provide universal usernames/IDs for Solid apps, and to refer to unique Agents (people, organizations, devices). See also the [WebID interoperability notes](http://www.w3.org/2005/Incubator/webid/wiki/Identity_Interoperability) for an overview of how WebID relates to other authentication and identity protocols.

* WebIDs, when accessed, yield [WebID Profile](http://www.w3.org/2005/Incubator/webid/spec/identity/#dfn-webid_profile) documents (in Turtle and other RDF formats).

* The [FOAF vocabulary](http://xmlns.com/foaf/0.1/) is used both in WebID profiles, and in specifying Access Control lists (see below).

* Authentication (for logins, page personalization and more) is done via the [WebID-TLS protocol](http://www.w3.org/2005/Incubator/webid/spec/tls/). WebID-TLS extends WebID Profiles to include references to the subject's [public keys](https://en.wikipedia.org/wiki/Public-key_cryptography) in the form of [X.509 Certificates](https://en.wikipedia.org/wiki/X.509), using [Cert Ontology 1.0](http://www.w3.org/ns/auth/cert) vocabulary. The authentication sequence is done using the [HTTP over TLS](https://tools.ietf.org/html/rfc2818) protocol. Unlike normal HTTPS use cases, WebID-TLS is done without referring to [Certificate Authority](https://en.wikipedia.org/wiki/Certificate_authority) hierarchies, and instead encourages host server-signed (or self-signed) certificates.

* In Solid, certificate creation is typically done in the browser using the HTML5 [keygen element](http://www.w3.org/TR/html5/forms.html#the-keygen-element), to provide a one-step creation and certificate publication user experience.

* Authorization and access lists are done using [Basic Access Control ontology](http://www.w3.org/ns/auth/acl) (see also the [WebAccessControl wiki page](http://www.w3.org/wiki/WebAccessControl) for more details).

* Solid uses the [Linked Data Platform (LDP)](http://www.w3.org/TR/ldp/) standard (see also [LDP Primer](http://www.w3.org/TR/ldp-primer/)) extensively, as a standard way of reading and writing generic Linked Data resources.

## How Solid Works

Solid applications are somewhat like multi-user applications where instances talk to each other through a shared filesystem, and the Web is that filesystem.

1. The [LDP specification](http://www.w3.org/TR/ldp/) defines a set of rules for HTTP operations on Web resources, some based on [RDF](http://www.w3.org/RDF/), to provide an architecture for reading and writing Linked Data on the Web. The most important feature of LDP is that it provides us with a standard way of RESTfully writing resources (documents) on the Web, without having to rely on less flexible conventions (APIs) based around sending form-encoded data using POST. For more insight into LDP, take a look at the examples in the LDP [Primer document](http://www.w3.org/TR/ldp-primer/).

2. Solid's basic protocol is REST, as refined by LDP with minor extensions. New items are created in a *container* (which could be called a collection or directory) by sending them to the container URL with an HTTP POST or issuing an HTTP PUT within its URL space. Items are updated with HTTP PUT or HTTP PATCH. Items are removed with HTTP DELETE. Items are found using HTTP GET and following links. A GET on the container returns an enumeration of the items in the container.

3. Servers are application-agnostic, so that new applications can be developed without needing to modify servers. For example, even though the [LDP 1.0](http://www.w3.org/TR/ldp/) specs contains nothing specific to "social", many of the [W3C Social Work Group](http://www.w3.org/Social/WG)'s [User Stories](http://www.w3.org/wiki/Socialwg/Social_syntax/User_Stories) can be implemented using only **application logic**, with no need to change code on the server. The design ideal is to keep a small standard data management core and extend it as necessary to support increasingly powerful classes of applications.

4. The data model is RDF. This means the data can be transmitted in various syntaxes like [Turtle](http://www.w3.org/TR/turtle/), [JSON-LD](http://www.w3.org/TR/json-ld/) (JSON with a "context"), or [RDFa](http://www.w3.org/TR/rdfa-primer/) (HTML attributes). RDF is REST-friendly, using URLs everywhere, and it provides **decentralized extensibility**, so that a set of applications can cooperate in sharing a new kind of data without needing approval from any central authority.

## How to Contribute to this Repo

To contribute to Solid development, and to bring up issues or feature requests, please use the following workflow:

1. Have a question or a feature request or spot a possible issue? [Open an issue on solid/solid](https://github.com/solid/solid).

2. Have an issue with the *Solid Specification*? Open an issue on [solid/solid](https://github.com/solid/solid) anyway. And then, as a result of discussion, if it's agreed that it is actually a Solid Specification issue, it will be moved to `solid-spec`.

3. The individual [solid/solid](https://github.com/solid) issues can coordinate and track component/dependent issues on the various affected Solid servers, apps, and so on.
  
## How to Join Our Community

Here's some ways to engage with the Solid Community:

#### 1. Solid Forum
- **Link:** https://forum.solidproject.org/
- **Description:** If you are getting familiar with Solid or building an app, this is the place to ask and speak to others in the Solid Community.

#### 2. Solid Chat 
- **Link:** https://gitter.im/solid/chat
- **Description:** If you have technical questions about the Solid Spec, Solid Core or Building Solid Apps, then come hang out in here! Also please help us out by assisting others with technical questions when you can!

## How to Learn More

* [Official Solid Website »](https://solidproject.org/)
* [Frequently Asked Questions »](https://solidproject.org/faqs).
* [Newsletter »](https://solidproject.org/newsletter)
* [Press »](https://solidproject.org/press)
* [Events »](https://solidproject.org/events)

## Contributors

Last, we'd like to thank our [Contributors](https://github.com/solid/solid/blob/master/CONTRIBUTORS.md). 
None of this would be possible without their help. 💜