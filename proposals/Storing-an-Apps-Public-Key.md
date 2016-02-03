Discussion at [Issue 47](https://github.com/solid/solid/issues/47) and on gitter.
Also see the links to [related issues](https://github.com/solid/solid/wiki/Storing-an-Apps-Public-Key#related-issues) section at the end of this document.

# Use Case

I have a simple foaf-browser application that can produce a public key using the JS WebSignature API and that can then be used to authenticate to the server using [Http-Signature](https://github.com/solid/solid-spec/issues/52) ((but this issue would be the same were I to use webid-rsa)). I can't really test it without saving the public key to the server. So I need a space for the application to save that key which can then identify the application/user.

This use case comes in two stages:

### A. Save Key

0. Create the public/private key in the browser using WebCrypto
1. find the LDPC workspace (from the foaf profile) where each application can create its storage LDPC
2. create  an LDPC there which it can control
3. initially just place its public key in there so that it can authenticate as that app to that workspace
4. Be able to set the ACLs there for the app, so that for example only apps from that origin when used by that user are able to change the key there.

The application running in that browser will be able to remember its storage space by saving the key in IndexDB or LocalStorage.

### B. Re-Discovery

Later if the same application runs in a different browser it will then need to start the procedure from scratch to find that storage space LDPC used by the previous browser. It will also need to be able to add a new key for that browser, and be given the same rights there as the app running in the first browser. How should that work?

# Protocol Sequence

Here is an initial idea of how this should work. __Please provide feedback__!!
The ontologies are currently very likely used wrong, and other things may have already been decided elsewhere that I (bblfish) am not aware of.


## Part A: Saving the Key


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

## B. Re-discovery

# Related Issues
 
* see [issue 53: wac:origin](https://github.com/solid/solid/issues/53) 
* [issue 52 of solid spec on Http-Signatures](https://github.com/solid/solid-spec/issues/52)
* instead of regexes it seems using `defaultForNew` may be better. See
 * [discussion on gitter at 8pm on Sunday 31 Jan 2016](https://gitter.im/solid/chat?at=56ae6a306b6468374a0a9ce0)
 * [issue 33: Name "defaultForNew" doesn't agree with semantics](https://github.com/solid/solid/issues/33)
 * [acl inheritance page on solid-spec](https://github.com/solid/solid-spec/blob/master/acl-inheritance.md)
 * for a defence of [Regexes in ACLs](https://github.com/solid/solid/wiki/Regexes-in-ACLs)
 