# Solid Notifications

Solid notifications are intended to announce social interactions and data. A Solid notification is an [LDP RDF Source](https://www.w3.org/TR/ldp/#dfn-linked-data-platform-rdf-source) which is submitted to and resides in an [LDP Container](https://www.w3.org/TR/ldp/#ldpc), i.e., a [Solid inbox](inbox.md).

The contents of a notification is application specific. We derive two core principles from this:

1. a notification can contain arbitrary data
2. a notification is vocabulary agnostic

Applications can create notifications to use any vocabulary to describe its contents in any shape and form. Multiple notifications can be contained inside a single resource or span across different resources. A single resource with multiple notifications may be suitable for `HTTP PATCH`, whereas multiple resources containing a single notification may be suitable for `HTTP POST` or `PUT`.


## Discovery
As notifications are contained in inboxes, a Solid inbox container has `ldp:contains` relations to each notification resource. If a [Solid outbox](outbox.md) is employed, it may also contain references to the notification.


## Notification integrity
Adhering to the integrity rules of an inbox notification e.g., for the purpose of a verification criteria or discovery is further described in [Solid inbox](inbox.md#shapes-constraint), is the responsibility of applications which submit and verify them.


## Enrichment
Although providing enriched (meta)data is voluntary, provenance level data like the date in which the request was submitted, who submitted, and its license, can be purposed towards the verification process as well as for displaying.


## Authorization
The owner of the Solid inbox typically sets its ACL rules and thereby for the notifications it contains. For example, notifications may be set for public read and append so that applications can use a FYN approach to discover and display the contents of a notification, as well as submit new notifications. Further, the ACL settings for the inbox may determine which agents can send notifications.


## Notification types
Here we exemplify some types of notifications:

### Singleton pattern
The singleton pattern is the simplest notification in terms of its data payload. It contains the most essential data which makes up a notification e.g, Alice liked an article:
```
<http://example.org/profile/card#alice> :liked <http://example.net/article> .
```

### RDF Statements
Making statements about statements; a statement in which each subject, property, object of a triple is described with its own triple:
```
<http://example.org/inbox/abc123> a rdf:Statement ;
  rdf:subject <http://example.org/profile/card#alice> ;
  rdf:property :liked ;
  rdf:object <http://example.net/article> .
```
This approach is useful if there is a need to further extend the statement's description or refer to it as a whole.

### Qualified relations
A particular grouping of triples which describe and qualify the relations, i.e., the vocabulary in use typically gives meaning to the statements when consumed as a whole. The examples below tend to refer to external resources where the body of the content can be discovered.

ActivityStreams:
```
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix as: <http://www.w3.org/ns/activitystreams#> .
<> a as:Announce ;
    as:object <https://example.org/26d8c1> ;
    as:context <http://www.w3.org/ns/oa#hasTarget> ;
    as:target <https://example.net/article#introduction> ;
    as:updated "2016-02-02T18:58:50.719Z"^^xsd:dateTime ;
    as:actor <http://example.org/profile/card#alice> .
```

Semantic Pingback:
```
@prefix pingback: <http://purl.org/net/pingback/> .
<> a pingback:Request ;
    pingback:source <http://example.net/foo/abc123> ;
    pingback:target <http://example.org/article/index> .
```

### Any data
This is typically in cases where an application may want to notify with any information. A notification may contain the complete payload and not necessarily refer to the origin data (external resource).
```
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix sioc: <http://rdfs.org/sioc/ns#> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
<> a sioc:Post ;
  dcterms:created "2015-12-23T16:44:21+00:00"^^xsd:#dateTime ;
  dcterms:title "New blog post" ;
  sioc:content "Lorem ipsum dolor sit amet, consectetur adipiscing elit..." ;
  sioac:has_creator <#author> .

<#author> a sioc:UserAccount ;
  sioc:account_of <../profile#me> ;
  sioc:avatar <../avatar.jpg> ;
  foaf:name "Alice" .
```
