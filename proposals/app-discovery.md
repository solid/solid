# Solid application configuration/preferences discovery (Draft)

The following describes app configuration discovery in the Solid framework 
using HTTP link following aka "follow your nose". This should not be 
confused with [application data discovery](https://github.com/solid/solid/tree/master/proposals/data-discovery.md)

## Starting point -- WebID

### Fetching the Profile

The public profile is what you get when you look up someone's WebID directly.
Strip off any hash and localid part. For example.

```
  https://example.databox.me/profile/card#me    ->     https://example.databox.me/profile/card
```

The starting point of discovery is a 
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

## Discoverability

Once a complete view of the profile has been created, applications will follow
links to discover where relevant configuration/preferences data is located.

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
