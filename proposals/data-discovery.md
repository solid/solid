# Solid application data discovery (Draft)

The following describes application data discovery in the Solid framework
using HTTP link following aka "follow your nose". This should not be
confused with [application preferences/configuration discovery]
(https://github.com/solid/solid/tree/master/proposals/app-discovery.md).

## Starting point -- WebID

### Fetching the Profile

The public profile is what you get when you look up someone's WebID directly.
Strip off any hash and local id (i.e. #me) part. For example:

```
https://example.databox.me/profile/card#me -> https://example.databox.me/profile/card
```

The starting point of Solid application data discovery is a user's
[WebID](http://www.w3.org/2005/Incubator/webid/spec/identity/), which is a hash
based URI, typically denoting a (FOAF) Agent. From this profile the user's
storage can be found (discovery). The Profile typically contains public name
and contact information about someone, and pointers to public data used by
various apps.

When an application dereferences the public profile, it should also fetch any

* owl:sameAs
* rdfs:seeAlso
* space:preferencesFile

links it finds in the public profile document. (one level deep)

The preferencesFile is a private file that is linked from the main WebID
profile, and contains miscellaneous data that does not belong in the public profile.
In general, both the public profile and the preferencesFil will contain triples
that share the same subject -- i.e the user's WebID.

## Discoverability

Once a complete view of the profile has been created, applications will follow
links to discover where relevant data is located in order to read and write
data there.

### Type registry configuration

The type registry is typically a document holding resources that register a
specific resource type and map it to a location on the user's data space.

**Note:** The Type Registry is mainly intended as a Library discovery mechanism.
Recommend that coarse-grained library types are registered (usually types that
match containers as opposed to every RDF Class written by an app)
-- e.g. `vcard:AddressBook` and `sioc:Blog` as opposed to `vcard:Contact` and
`sioc:Post`.

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
<#me>  space:typeIndex  </settings/publicTypeIndex.ttl>.
```

In the preferencesFile:
```
<#me>  space:typeIndex  </settings/privateTypeIndex.ttl>.
```

The TypeIndex resource `/settings/privateTypeIndex.ttl` will contain things like:
```
<#ab09fd>  a solid:TypeRegistration;
    solid:forClass vcard:AddressBook;
    solid:instance </contacts/>.
```

Optional:

- An app may register a defaultApp URI when creating a data container.
- When the user opens that container in the browser, a meshlib-like (thin)
app can be loaded first, which in turns fetches the meta file and displays
the registered app.


### Storage Discovery

* Starting Point: WebID
* Type: [pim : storage](http://www.w3.org/ns/pim/space#storage)

For example, for a given WebID (https://example.databox.me/card#me),
the corresponding storage location URI is https://example.databox.me/.

```
<#me>
space:storage <.> .
<http://www.w3.org/ns/pim/space#storage> <../> ;
```
