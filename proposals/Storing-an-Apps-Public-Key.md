Discussion at [Issue 47](https://github.com/solid/solid/issues/47) and on gitter.
Also see the links to [related issues](https://github.com/solid/solid/wiki/Storing-an-Apps-Public-Key#related-issues) section at the end of this document.

# Use Case

I have a simple foaf browser/editor app that can produce a public key using the JS WebSignature API and that can then be used to authenticate to the server using [Http-Signature](https://github.com/solid/solid-spec/issues/52) ((but this issue would be the same were I to use webid-rsa)). To test the HTTP-Signature authentication the public key should be saved in a publicly visible resource.

The application should perhaps also recognise if it has published a key in a different browser and try to tie them together somehow. This may be done by tying the key to the WebID.

# Index

There are a number of ways of doing this which are not exclusive:

* [P1](#p1-publish-key-in-public-space): Publish the key in a public space (perhaps using a distributed hash table protocol)
* [P2](#p2-create-a-specialised-indirect-container): Place the key in a specialised [(In)](https://www.w3.org/TR/ldp/#ldpic)[direct](https://www.w3.org/TR/ldp/#ldpdc) Container that also adds the link from the webid to the key
* [P3](#p3-place-the-key-in-a-specialised-basic-container): Place the key in a Specialised Basic Container
* [P4](#p4-saving-the-key-to-the-applications-space): Create a Container for the application in the user's application space and save the key there


# P1. Publish key in public space

The public space could be some specialised LDP container that accepts only public keys, or a Distributed Hashtable such as:
* DigitalBazaar's [A Decentralized Hashtable for the Web](http://opencreds.org/specs/source/webdht/)
* [IPFS](https://ipfs.io/)
* many others

### Advantages:
* Bootstrapping: Given such a public space the advantage would be that the user could publish a public key without even needing to have his personal space. So this may be a good way to bootstrap the use of an application. 
* Privacy: the advantage that there is no way to tie the key to any individual, as the appearance of the key somewhere will have no bearing on the possible identity of the user. Ie one won't be able to deduce from the key's being published at a domain, that the author of the domain is somehow related to the use of the key, if the key does not point to the user of course).

### Considerations:
* spam: the public space has to be spam proof, as there would presumably be no authentication required to use it, and it could be filled in quickly. 
* linking keys: If the user changes application he will have a lot of trouble finding his public key unless the public key is linked to from his WebID, or from the application space linked to from his WebID.

### Questions:
* How does the application then later link to the WebID profile without needing all rights to that document?
* One would need a specific relation to point to such a container
* The container may need to specify that it limits itself to specific public keys perhaps using [SHACL](https://www.w3.org/TR/2016/WD-shacl-20160128/)


# P2. Create a specialised (In)Direct Container

See [Indirect](https://www.w3.org/TR/ldp/#ldpic) and [direct Container](https://www.w3.org/TR/ldp/#ldpdc) Container in LDP spec.

An LDP (In)direct Container's allow have side effects such as allowing new relations to be added elsewhere.
It would look something like this:

```Turtle
<> a ldp:IndirectContainer;
  ldp:membershipResource </card#me>;
  ldp:hasMemberRelation cert:key;
  ldp:insertedContentRelation foaf:primaryTopic.
``` 

By POSTing your public key there like this

```HTTP
POST </keys/>
Content-Type: text/turtle
...

<> foaf:primaryTopic <#> .
<#> a cert:RSAPublicKey;
    cert:exponent 65537 ;
    rdfs:comment "KeyChain OSX laptop";
    cert:modulus "DAB9D1E941F6F85A0863169D0DB6328D1D4A15A71DFFE3D4F4D08752A52FB1454D7358E4A5ECF3501E3924BC0252F3004B0BB21A0D6B64CA053F0FBCB5A54EC93EBE2DC9B91E4C432B827884C4CC2AD8A102B46D2A2017BF45D9D4C88A564D420234484A1B2E446DBB4CD438E79C2466CE310F327773A779D24ED7B60A05A618B984757B946D67BA79F2E064E6AED38BD6559CE7FC9502720823D56DB1C034099367D7DB27B6BDAFDA8CC48347133F4A14675F675FB484CE32DF66C11A3638FA84D5BE69B1A6F238115DEF9B0F79BB25C0CB7E4A39459A0829B1FD35C0D1DBDD60F9C679D89415ED7EA41EB02FBC016FC0E792CB9698C9F4DB842CDAD5B5F5C9"^^xsd:hexBinary .              
```

It would create a new resource in the container and return the link for the resource, as with an ldp:BasicContainer, but it would also add to the `</card>` LDPR the following relation:

```Turtle
<#me> cert:key </keys/key20#> .
```

### Advantages: 
* very simple to use
* Easy Linking to WebID: A specialised InDirect container would allow the application to add the key to the WebID profile whilst publishing the link without the application being given any special rights to the WebID profile.
* security: the app that is allowed to POST to the `</keys/>` container may not have rights to edit the profile. The profile may have the following ACL, only allowing one application to edit it:

```Turtle
[] acl:agent [ a acl:JSUserAgent;
           acl:origin <https://ProfileEditor.rww.io/>;
           acl:user <card#me>
           ];
   acl:mode acl:Read, acl:Write;
   acl:accessTo <card> .
```

(see [issue 53: wac:origin](https://github.com/solid/solid/issues/53))
Whereas the `</keys/>` LDPC would have the ACL, allowing any app to create new resources in it.

```Turtle
[] acl:agent <card#me>;              
   acl:mode acl:Read, acl:Append;
   acl:accessTo </keys/> .
```

(we don't want any app to edit the resource itself, to change the meaning of what is posted there).

### Considerations:
* As the LDPC has to be on the same server as the profile (so that the server can update the profile) this has less strong privacy features as publishing in a public space
* Requires an (In)direct Container
* The container will need to specify that it limits itself to specific public keys perhaps using [SHACL](https://www.w3.org/TR/2016/WD-shacl-20160128/)

### Questions/Todo:
* One would need a specific relation to point to such a container (as in previous section) as in P1
* The container may need to specify that it limits itself to specific public keys perhaps using [SHACL: Shapes and Constraints Language](https://www.w3.org/TR/2016/WD-shacl-20160128/) as in P1
* Use of (In)Direct containers is still quite new:
  * How are they set up? Presumably they can only be set up by an actor able to write to the resource that may have relations added to it...
* @dmitrizagidulin [asked on gitter](https://gitter.im/solid/chat?at=56b61b38a44cf16b6de0359c): "how would one indicate from the `<card>` resource that it can be written to by other agents via the indirect container"? That would be important for the owner of `<card>` to be able to remember when looking at it, or else he may forget this other way of adding information to the resource. (  That may require one to think of (In)Direct Containers as Actors, and give them rights in the ACL, but very specific rights that depend on their type. )

# P3. Place the key in a Specialised Basic Container

This is similar to P2, but instead of having a Direct or Indirect Container we have a relation
call it `solid:keyContainer` that points from the WebID to a container that contains only keys

```Turtle
<#me> solid:keyContainer </keys/> .
```

Such that whenever a key is posted to the container 

```Turtle
POST </keys/>
Content-Type: text/turtle
...

<> foaf:primaryTopic <#> .
<#> a cert:RSAPublicKey;
    cert:exponent 65537 ;
    rdfs:comment "KeyChain OSX laptop";
    cert:modulus "DAB9D1E941F6F85A0863169D0DB6328D1D4A15A71DFFE3D4F4D08752A52FB1454D7358E4A5ECF3501E3924BC0252F3004B0BB21A0D6B64CA053F0FBCB5A54EC93EBE2DC9B91E4C432B827884C4CC2AD8A102B46D2A2017BF45D9D4C88A564D420234484A1B2E446DBB4CD438E79C2466CE310F327773A779D24ED7B60A05A618B984757B946D67BA79F2E064E6AED38BD6559CE7FC9502720823D56DB1C034099367D7DB27B6BDAFDA8CC48347133F4A14675F675FB484CE32DF66C11A3638FA84D5BE69B1A6F238115DEF9B0F79BB25C0CB7E4A39459A0829B1FD35C0D1DBDD60F9C679D89415ED7EA41EB02FBC016FC0E792CB9698C9F4DB842CDAD5B5F5C9"^^xsd:hexBinary .              
```

this creates a new resource e.g. `</keys/key12>` and adds the following to the `</keys/>` container

```Turtle
<> ldp:contains <key12> .
```

The meaning of `solid:keys` could be expressed by the following SPARQL rule:

```
CONSTRUCT {
  ?id cert:key ?key .
} WHERE {
  ?id solid:keyContainer ?ldpc .

  GRAPH ?ldpc {
     ?ldpc ldp:contains ?ldpr .
  }
  GRAPH ?ldpr {
     ?ldpr foaf:primaryTopic ?key .
     ?key a cert:Key . #<-- this should be inferred 
  }
}
```

(does this say what I mean?)

As with P2 access to the container should be restricted to the webid owner.

### Advantages

* As opposed to P2, this does not require (In)Direct Containers, and so
 * Basic Containers are much more easy to implement than (In)Direct Containers
 * the security issue mentioned in P2 does not apply
* The user's profile can remain much cleaner, by having only one link to a container that holds all the keys 

### Considerations

* Given that all keys published in this container belong to the user it is much easier to trace back each key to the user, so privacy will not be as strong as P1, when published on an anonymous server or in a Distributed Hash Table.
* The container may need to specify that it limits itself POSTS to specific public keys using something like [SHACL](https://www.w3.org/TR/2016/WD-shacl-20160128/)

### Questions/Todo

* Can such a pattern always replace the (In)Direct Containers?
* What type of rule is the SPARQL rule expressed above? It seems to be a mixture of ontological reasoning and a rule about how to merge liked data expressed in different graphs and combined them into one.

# P4. Saving the key to the application's space

Here the application needs to find the user's storage space or create it and save the key there.

### Advantages:
* The application very likely needs to have a storage space to keep preferences for the application, so this needs to be covered in any case
* The storage space can be limited by the user for that app (would need a vocabulary)
* The application can organise that space as it wishes, so there is no need to restrict what it puts there

### Considerations:
* How does the application add the relation to the key from the WebID if it desires to do so? (Without the user giving the application the right to override all information in the profile?)

### Comparative advantage 

* P1 may require a public space to publish keys (though it could use the same relation as P2)
* P1 may require implementing or working with a new DHT protocol
* P1 & P2 potentially require a Shapes and Constraints language
* P2 requires a direct or indirect container, which I have not yet implemented
* P2 does not give much additional privacy 

## Sequence

Here is an initial idea of how this should work. __Please provide feedback__!!
The ontologies are currently very likely used wrong, and other things may have already been decided elsewhere that I (bblfish) am not aware of.

### 1 find workspace

Find ldpc "workspace" by following `ws:storage` link in webid profile. If it contains something like this:

```Turtle
@prefix ws: <https://www.w3.org/ns/pim/space#> .
<#me> ws:storage </apps/> .
```

#### Questions

1. Is ws:storage the correct relation here? 
  * If not do we have correct one?
  * Do we have to create one?

### 2. Create App container.

Two options:
 1. `POST` to `</apps/>` with `Slug` header "rww-foaf"
 2. `PUT` to `</apps/rww-foaf/>` with correct headers (pending resolution of [issue 37](https://github.com/solid/solid/issues/37))

#### Questions

##### 1. How does an app decide on a name for the LDPC?
 One wants that it be both humanly readable and does not clash with other existing ones? 
  * One answer may be to name each LDPC after the origin of the app
    * If so it cannot be up to the client to name the application. It must be the server that does it, and so this requires a special LDPC type that has this property. 
      * con: We want to avoid usually having to implement server specific behavior such as this.
    * This would require one to think about how enforceable this actually is. Setting the Origin of an application is easy in HTTP. If the user does authenticate with WebID then one would have a guarantee at least
that the browser set this up correctly.
  * Another option is just to let the client choose a short name and then require it to describe itself.
    * Perhaps the description can be via reference to a description at the origin's location.
 
##### 2. How does the application check that it has not already created a space for itself?

  * If the container is named after the origin of the app, then this is quite easy.
  * If the container has a short name then a document with the descriptions of the apps has to be searched.
    * If there is one such document that all apps use, then it is easy for one application to overwrite through bad code or maliciously the information of another application.
    * If each container has its own document then the previous issue does not arise, but the problem of searching through a potentially large list of containers could be problematic. 
      * Unless a `SEARCH` type functionality is made available. See [Issue 46: Document Query Use Cases](https://github.com/solid/solid/issues/46) 
      * Unless each LDPC is visible only to the origins that made them, which would be enabled by a container that followed [Issue 29: Current Container Listing mechanism does not support hidden resource use case](https://github.com/solid/solid/issues/29). 

##### 3. How do we make sure not everyone can write to the container?
  One could do this by setting a write restriction on the Container to only allow the user to Write there.
```Turtle
   [] acl:accessTo <.>; 
      acl:mode acl:Read, acl:Write;  
      acl:agent <card#i> .
```
  Other authentication protocols could be listed, or assumed by the description of `<card#i>`, say if it lists an OpenId and the server allows openId authentication too (using `WWW-Authenticate: OpenID` if that exists)
 
##### 4. How does one specify the ACLs of the newly created resource? 

  * The Default could be that the user identified by the WebID can read/write to the resource. This would allow any application to write to the space though, so the application should right after creation edit the ACL of the contained elements and restrict those to be readable only by the application as described by [issue 53: acl:origin](https://github.com/solid/solid/issues/53):
```Turtle
  [] acl:accessTo <.>;
     acl:mode acl:Read, acl:Write;
     acl:agent [ a acl:JSUserAgent;
                acl:origin <https://apps.rww.io/>;
                acl:user </card#i> ] .
```
 * to specify the default one may use (note that none of these actually allows one to specify the `acl:origin` as that would be different for each caller)
   * defaultForNew: [issue 33](https://github.com/solid/solid/issues/33)
   * wac:include: [Discussed in Regexes and ACLs](https://github.com/solid/solid/wiki/Regexes-in-ACLs#algorithm)
  

### 3. Create key doc

To create the application instance file, we have two options as above:
 1. `POST` to `</apps/rww-foaf/>` with Slug "key"
 2. `PUT` to `</apps/rww-foaf/key>` (pending resolution of [issue 37](https://github.com/solid/solid/issues/37))
The content of either would be minimally along the lines of this:

```Turtle
@prefix cert: <http://www.w3.org/ns/auth/cert#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<#> a cert:RSAPublicKey; 
    rdfs:comment "Key generated by User Agent: Chrome on OSX";
    cert:exponent 65537 ;
    cert:modulus "DAB9D1E941F6F85A0863169D0DB632...."^^xsd:hexBinary .
``` 

This is actually all that is needed to authenticate with HTTP Signatures as that only requires the URL of a key.
                 
### 4. Set ACL for Key

Assuming the application is published at the origin `https://rww-foaf.github.io/` 

The ACL for the all content in `</apps/rww-foaf/>` would be  

```Turtle
@prefix acl: <http://www.w3.org/ns/auth/acl#> . 

[] acl:accessToClass [ acl:urlPattern [ acl:base <.>; acl:match "**" ];  
   acl:mode acl:Write; 
   acl:agent [ acl:JSUserAgent; 
               acl:user [ cert:key <key#> ];
               acl:origin <https://rww-foaf.github.io/> 
           ]. 
```


# Related Issues
 
* see [issue 53: wac:origin](https://github.com/solid/solid/issues/53) 
* [issue 52 of solid spec on Http-Signatures](https://github.com/solid/solid-spec/issues/52)
* instead of regexes it seems using `defaultForNew` may be better. See
 * [discussion on gitter at 8pm on Sunday 31 Jan 2016](https://gitter.im/solid/chat?at=56ae6a306b6468374a0a9ce0)
 * [issue 33: Name "defaultForNew" doesn't agree with semantics](https://github.com/solid/solid/issues/33)
 * [acl inheritance page on solid-spec](https://github.com/solid/solid-spec/blob/master/acl-inheritance.md)
 * for a defence of [Regexes in ACLs](https://github.com/solid/solid/wiki/Regexes-in-ACLs)
 