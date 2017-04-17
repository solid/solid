# Solid ripples

Solid "ripples" is a pingback method for handling request notifications where a target resource refers to the source resource with a specific relationship - a triple statement. A user (or an agent) essentially informs the source about this statement because it may be of interest to them. For example, Alice writes an article on her site. Bob then reads this article and comments about it, linking back to Alice's original post. Using pingback, a Solid aware application can notify Alice's post about this link, and Alice's software can then include this information on her site.

## Requirements
For a successful pingback to happen, there are some requirements for the container, source and target resources:

### Pingback container
Any resource can potentially receive a pingback, and the relation for it may be declared anywhere. It is up to the receiver to decide the location of the container since there are different use cases. For instance, it is not only the documents which may receive a pingback but any fragment within, e.g., a sentence in a paragraph may be annotated, liked, or replied to. The selection for what can potentially receive a pingback can also differ between media types e.g., a segment in a photo may be tagged, and the owner of the photo gets notified.

All containers and workspaces may have a `rel="pingback:to"` pre-set. The containers may also want to provide a triple in which it can give further instructions (human and/or machine-readable) on how a claim made to that container may be verified.

If the user wants to refine pingback reciept for specific containers/resources, they can set preferences to point to different `rel="pingback:to"` which may have different ACLs set, allowing blacklisting or whitelisting or total prevention of responses, if desired.

A user should have a link to a `pingback:to` from their WebID, for if someone creates a relationship with the user directly, rather than one of their resources.

### Source resource
`http://example.org/article/index` has relation `pingback:to` (`http://purl.org/net/pingback/to`) to `http://example.org/article/pingback/` (which is of type `ldp:Container`). The relation may be declared at one of the following places:
  * HTTP Link header e.g., `Link: <http://example.org/article/pingback/>; rel="pingback:to"`
  * Metadata resource e.g., `http://example.org/article/index,meta` in Turtle: `<http://example.org/article/index> <http://purl.org/net/pingback/to> <http://example.org/article/pingback/> .`
  * Resource body e.g., HTML link element: `<link rel="pingback:to" href="http://example.org/article/pingback/"/>`, HTML+RDFa:`<a about="http://example.org/article/index" rel="pingback:to" href="pingback/">pingback</a>`, Turtle: same as above metadata example.
  * Resource's container e.g., `http://example.org/article/`

For discovery, if the relation is not found at any of the places above, the sender continues up the hierarchy until the `pingback:to` relation is found or halted.

### Target resource
The resource at `http://example.net/foo/abc123` has relation to source resource at `http://example.org/article/index`. An example in RDF Turtle:
```
@prefix as: <http://www.w3.org/ns/activitystreams#> .
<http://example.net/foo/abc123>
    as:inReplyTo <http://example.org/article/index> ;
    as:content "Cogito ergo sum." .
```

## Notification
Notifying the pingback container about target's statement is done by making a POST or PUT request to the container `http://example.org/article/pingback/` and reside at `http://example.org/article/pingback/abc123`. It is essentially a request given to the receiver so that they can decide what to do with the claim. An example in RDF Turtle:

```
@prefix pingback: <http://purl.org/net/pingback/> .
<> a pingback:Request ;
    pingback:source <http://example.net/foo/abc123> ;
    pingback:property <http://www.w3.org/ns/activitystreams#inReplyTo> ;
    pingback:target <http://example.org/article/index> .
```

Although providing enriched (meta)data is voluntary, provenance level data like the date in which the request was submitted, who submitted, and its license, can be purposed towards the verification process as well as for displaying.

The sender may also want to set their own credentials at the corresponding ACL `http://example.org/article/pingback/abc123,acl`. This is so that only them or the article's author may change the pingback request. If the sender is not authorized to write to the container, processing stops.


## Verification
1. There exists a client with the ability to verify the existance of relationships between resources, whether all that were ever sent, or a subset that this particular client is interested in.
2. The client locates the list of asserted relationships via the `rel="pingback:to"` Link Header of resources/containers/users it is interested in. Presumably the user has told it what to be interested in, or it has a predefined list of types of thing it's interested in because it's a client with a specific purpose, and can follow links to more if need be.
3. For each pingback request of interest, the client:
  * 3.1 `GET`s the `pingback:target` value and verifies that it exists.
  * 3.2 `GET`s the `pingback:source` value and verifies that it exists, and containes a link to the `pingback:target` value, with the value of `pingback:property`.
  * 3.3 If both of these conditions are met, the client may optionally proceed to check against other criteria as desired before accepting.
4. Once verified, the client may want to `HTTP PATCH` the pingback request and further enrich with the validation information e.g., when it was validated, by whom, under which conditions.
