# Solid Application Data Discovery

The following describes application data discovery in the Solid framework
using HTTP link following aka "follow your nose". This should not be
confused with [Application Configuration
Discovery](https://github.com/solid/solid/tree/master/proposals/app-discovery.md)
or [Storage
Discovery](https://github.com/solid/solid-spec/blob/master/solid-webid-profiles.md#storage-discovery).

**Note:** The Type Registry is mainly intended as a Library discovery mechanism.
We recommend that coarse-grained library types are registered (usually types
that match containers as opposed to every RDF Class written by an app).

Specifically, the Type Registry provides a way for a client application to
discover where a user keeps data relevant to this app, without either:

a. Prompting the user to select the location of every relevant instance or
  container, or
b. Scanning through the entire dataspace/root storage of the user.

For example, when a user installs a new Contacts Manager app, the app does not
need to scan all of the containers in a user's storage, looking for relevant
Contacts or AddressBook resources. Instead, it can query the Type Index registry
and discover only the resources and containers it cares about.

## Data Discovery Workflow

Solid client apps use the following workflow to discover where data resides in
a user's dataspace:

1. Start with the [WebID URI](https://github.com/solid/solid-spec#identity).
2. Use it to fetch the [WebID Profile
  Document](https://github.com/solid/solid-spec#profiles).
3. Parse the profile and load the other [Extended
  Profile](https://github.com/solid/solid-spec/blob/master/solid-webid-profiles.md#extended-profile)
  resources, which includes the Preferences file and any `owl:sameAs` and
  `rdfs:seeAlso` links.
4. From the Extended Profile [discover storage instances](https://github.com/solid/solid-spec/blob/master/solid-webid-profiles.md#storage-discovery)
5. For each storage, extract the registry links
  (the `solid:publicTypeIndex` predicate link to the public Listed Type Index
  registry, and the `solid:privateTypeIndex` predicate link to the private
  Unlisted Type Index registry).
6. Load one or both of the Type Index registry documents, as appropriate, and
  query them for the location of RDF Types that the app cares about.

Example snippet from document describing a Storage, with links to both type index resources:

```ttl
@prefix solid: <https://www.w3.org/ns/solid/terms#>.
@prefix pim: <http://www.w3.org/ns/pim/space#>.
# ...
<#storage>
    a pim:Storage;
    # Listed type index resource:
    solid:publicTypeIndex </settings/publicTypeIndex.ttl> ;  
    # Unlisted type index resource:
    solid:privateTypeIndex </settings/privateTypeIndex.ttl> .
```

## Type Index Registry

The Solid Type Index Registry consists of two Type Index resources, a
Listed Type Index (publicly readable by default) and an Unlisted Type Index
(readable by the owner only, by default). Each of those resources
contains registry entries that map a specific resource type to a location in
the user's data space.

### Type Index Registrations

The type index resources contain any number of statements of type
`solid:TypeRegistration` which map RDF classes/types to their locations in a
user's dataspace/root storage.

The registration entries map a type to a location using one of two predicates:

##### `solid:instance`
maps a type to an individual Solid *resource*, typically an index or a directory listing resource such as an Address Book.

##### `solid:instanceContainer`
maps a type to a Solid *container* which the client would have to list to get
the instances of that type.

An example Listed Type Index resource might contain:

```ttl
# Maps the type vcard:AddressBook to an index document
<#ab09fd> a solid:TypeRegistration;
    solid:forClass vcard:AddressBook;
    solid:instance </contacts/myPublicAddressBook.ttl>.

# Maps the type sioc:Post to a container
<#ab09cc> a solid:TypeRegistration;
    solid:forClass sioc:Post;
    solid:instanceContainer </posts/>.
```

Changing the status of a registration (from a Listed index to an Unlisted or
vise versa) involves *removing* that registration (typically via a SPARQL-based
HTTP PATCH) from the one and *adding* it (also via a PATCH) to the other.

### Listed Type Index

The Listed Type Index is intended for registrations that are *discoverable by
outside users and applications*. For example, think of a listed phone number in
a public phonebook, which contains publicly-discoverable mappings of people's
names to phone numbers and addresses.

The Listed (public) Type Index has the following properties:

* Is linked to from the WebID Profile using the `solid:publicTypeIndex`
  predicate
* Is created in `/settings/publicTypeIndex.ttl` by default
* Has a public-readable ACL by default
* Is of type `solid:ListedDocument`

Example Listed Type Index resource containing one registration entry:

```ttl
@prefix solid: <https://www.w3.org/ns/solid/terms#>.
# ...
<>
    a solid:TypeIndex ;
    a solid:ListedDocument.

<#ab09fd> a solid:TypeRegistration;
    solid:forClass vcard:AddressBook;
    solid:instance </contacts/myPublicAddressBook.ttl>.
```

### Unlisted Type Index

The Unlisted Type Index resource is intended for registrations that are private
to the user and their apps, for types that are *not* publicly discoverable.

The Unlisted (private) Type Index has the following properties:

* Is typically linked to from the Preferences file (or some other private
  section of an Extended Profile) using the `solid:privateTypeIndex` predicate
* Is created in `/settings/privateTypeIndex.ttl` by default
* Has a private ACL by default (readable only by the owner)
* Is of type `solid:UnlistedDocument`

Example Unlisted Type Index resource:

```ttl
@prefix solid: <https://www.w3.org/ns/solid/terms#>.
@prefix sioc: <http://rdfs.org/sioc/ns#>.
# ...
<>
    a solid:TypeIndex ;
    a solid:UnlistedDocument.

<#ab09cc> a solid:TypeRegistration;
    solid:forClass sioc:Post;
    solid:instanceContainer </personal-diary/>.
```
