# Solid Discovery (Draft)

The following describes discovery in the Solid framework using HTTP link
following aka "follow your nose".

### Starting point -- WebID

#### Fetching the Profile

The public profile is what you get when you look up someone's WebID directly.
Strip off any hash and localid part. For example.

```
  https://example.databox.me/profile/card#me    ->     https://example.databox.me/profile/card
```

The starting point of Solid discovery is a
[WebID](http://www.w3.org/2005/Incubator/webid/spec/identity/) user profile,
which is a hash based URI, typically denoting a (FOAF) Agent. From this profile
all of your storage can be found (discovery). The Profile typically contains
public name and contact information about someone, and pointers to public data
used by various apps.

When an application dereferences the public profile, it should also fetch any

* owl:sameAs
* rdfs:seeAlso
* space:preferencesFile

links it finds in the public profile document. (one level deep)

The preferencesFile is a private file that is linked from the main WebID
profile, and contains miscellaneous data not in your public profile. In
general, the same triples will be put in the public profile.

### Discoverability

Once a complete view of the profile has been created, applications will follow
links to discover where relevant data is located, in order to read and write
data there.

## Type registry configuration

The type registry is typically document that holds resources that register a
specific resource type and map it to a location on the user's data space.

Note: The Type Registry is mainly intended as a Library discovery mechanism.
Recommend that coarse-grained library types are registered (as opposed to every
RDF Class written by an app)

A typical Solid account will have a private type index, where applications will
register the top container type. For example, a contacts app will register the
type `vcard:AddressBook` and the instance (or data location) as `/contacts/` in
a new resource at `/settings/privateTypeIndex.ttl#ab09fd`.

If the user decides it wants to make public that she is using a certain
application, it can indicate this  by registering the same type in a public
index document at `/settings/publicTypeIndex.ttl#19da01`.

Both registry documents will be linked to from the user's main profile (and from
it, the preferences file):

In the public profile:
```
<#me>  space:TypeIndex  </settings/publicTypeIndex.ttl>.
```

In the preferencesFile:
```
<#me>  space:TypeIndex  </settings/privateTypeIndex.ttl>.
```

The TypeIndex resource will contain things like:
```
 <#r1>  a solid:TypeRegistration;
  solid:forClass   vcard:addressBook;
  solid:instance </contacts/book#this>.
```

### By application

A list (public or private) of

```
  <#me>  space:AppIndex  <byType>.
```

And then in there things like

```
 <#r5>  a solid:AppRegistration;
  solid:forApp  ghld:app-shedule
  solid:instanceIndex </polls/list.ttl>.
```

and then in `/polls/list.ttl` things like

```
<#i345> a solid:Instance;
  solid:forApp  ghld:app-shedule;
  dc:created 2012-12-09;
  solid:instance </polls/list3/congig#this>;
```

Alternative:

- An app may register a defaultApp URI when creating a data container.
- When the user opens that container in the browser, a meshlib-like (thin)
- app can be loaded first, which in turns fetches the meta file.

### Storage Discovery

#### Storage

* Starting Point: WebID
* Type: [pim : storage](http://www.w3.org/ns/pim/space#storage)

For example, for a given WebID (https://example.databox.me/card#me),
the corresponding storage location URI is https://example.databox.me/.

```
<#me>
space:storage <.> .
<http://www.w3.org/ns/pim/space#storage> <../> ;
```
