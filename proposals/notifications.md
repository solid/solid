> This is an archived, legacy repository that is no longer actively maintained. Please visit the [official Solid website](https://solidproject.org/) instead.

# Solid inboxes and notifications

**Note:** The main notification spec has moved to:
https://www.w3.org/TR/ldn/

Solid inboxes are repositories for incoming social interactions and data. A
Solid inbox is an [LDP Container](https://www.w3.org/TR/ldp/#ldpc) for Solid
notifications. A resource can have one or more inboxes in order to facilitate
different social interactions. For instance, user profile inboxes are typically
intended to receive notifications like messages from other profiles, or
application specific activities.

By default, Solid servers should create an inbox if the account also creates an
identity (i.e., a WebID profile). Alternative types of inboxes e.g., for
documents or data fragments, may be created at the discretion of applications
taking user's preferences into account.

Solid notifications are intended to announce social interactions and data. A
Solid notification is an [LDP Resource](https://www.w3.org/TR/ldp/#ldpr) which
is submitted to and resides in an Solid inbox.

The contents of a notification is application specific. We derive two core
principles from this:

1. a notification can contain arbitrary data
2. a notification is vocabulary agnostic

Applications can create notifications to use any vocabulary to describe its
contents in any shape and form. Multiple notifications can be contained inside a
single resource or span across different resources. A single resource with
multiple notifications may be suitable for `HTTP PATCH`, whereas multiple
resources containing a single notification may be suitable for `HTTP POST` or
`PUT`. If different access controls are desired for inboxes and notifications,
implementing a multiple notification approach with `HTTP POST` or `PUT` may be
preferable.


## Namespaces and document conventions
The following namespaces are used in this document:

|Prefix|Namespace
|------|---------
|solid|http://www.w3.org/ns/solid/terms#
|xsd|http://www.w3.org/2001/XMLSchema#
|dcterms|http://purl.org/dc/terms/
|sioc|http://rdfs.org/sioc/ns#
|foaf|http://xmlns.com/foaf/0.1/
|pingback|http://purl.org/net/pingback/
|as|http://www.w3.org/ns/activitystreams#
|schema|https://schema.org/

All RDF examples are written in Turtle syntax.


## Discovery

An inbox container must be discoverable via the `solid:inbox` property, and
should be instances of `solid:Inbox`. Each notification can be discovered via
`solid:notification` (or `ldp:contains` for LDP based applications) relations.
Notifications that are of LDP RDF Source should be instances of
`solid:Notification`. Although neither `solid:Inbox` or `solid:Notification` is
ultimately required to discover inboxes or notifications, it is good practice to
instantiate resources for the purpose of data integrity and reuse. An example:

```
<https://example.org/profile> solid:inbox <https://example.net/inbox/> .
<https://example.net/inbox/> a solid:Inbox .
<https://example.net/inbox/> solid:notification <https://example.net/inbox/abc123> .
<https://example.net/inbox/abc123> a solid:Notification .
```


## Notification integrity

Adhering to the integrity rules of an inbox notification e.g., for the purpose
of a verification criteria or discovery is the responsibility of applications
which submit and verify them. An inbox may be accompanied with constraint
information e.g., by employing the [Shapes Constraint
Language](https://www.w3.org/TR/shacl/)(SHACL) or [Shape
Expressions](https://www.w3.org/2001/sw/wiki/ShEx)(ShEx).


## Enrichment

Although providing enriched (meta)data is voluntary, provenance level data like
the date in which the request was submitted, who submitted, and its license, can
be purposed towards the verification process as well as for displaying.


## Authorization

The owner of the Solid inbox typically sets its ACL rules and thereby for the
notifications it contains. By default, when profile inboxes are created, inboxes
must be public append only as they are inherently private. Other types of
inboxes e.g., for blog posts, may be set to public read and append so that
applications can use a *follow-your-nose* approach to discover and display the
contents of a notification, as well as submit new notifications. Further, the
ACL settings for the inbox may determine which agents can send notifications.


## Notification types

Here we exemplify some types of notifications:

### Single statement

A single triple statement is the simplest notification in terms of its data
payload. It contains the most essential data which makes up a notification e.g,
Alice liked an article:

```
<http://example.org/profile/card#alice> :liked <http://example.net/article> .
```

This approach is useful if there is a need to further extend the statement's
description or refer to it as a whole.

### Qualified relations

A particular grouping of triples which describe and qualify the relations, i.e.,
the vocabulary in use typically gives meaning to the statements when consumed as
a whole. The examples below tend to refer to external resources where the body
of the content can be discovered.

ActivityStreams:
```
<> a solid:Notification , as:Announce ;
  as:object <https://example.org/26d8c1> ;
  as:context <http://www.w3.org/ns/oa#hasTarget> ;
  as:target <https://example.net/article#introduction> ;
  as:updated "2016-02-02T18:58:50.719Z"^^xsd:dateTime ;
  as:actor <http://example.org/profile/card#alice> ;
  schema:license <http://creativecommons.org/licenses/by-sa/4.0/> .
```

Semantic Pingback:

```
<> a solid:Notification , pingback:Request ;
  pingback:source <http://example.net/foo/abc123> ;
  pingback:target <http://example.org/article/index> .
```

### Any data

This is typically in cases where an application may want to notify with any
information. A notification may contain the complete payload and not necessarily
refer to the origin data (external resource).

```
<> a solid:Notification , sioc:Post ;
  dcterms:created "2015-12-23T16:44:21Z"^^xsd:#dateTime ;
  dcterms:title "New blog post" ;
  sioc:content "Lorem ipsum dolor sit amet, consectetur adipiscing elit..." ;
  sioac:has_creator <#author> .

<#author> a sioc:UserAccount ;
  sioc:account_of <../profile#me> ;
  sioc:avatar <../avatar.jpg> ;
  foaf:name "Alice" .
```
