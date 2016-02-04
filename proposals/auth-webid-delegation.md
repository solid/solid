### WebID Delegated Requests

A very interesting use case supported by Solid deals with the ability to
delegate certain requests to an agent (robot) representing a Solid server. For
example, instead of performing a lot of cross-origin requests in the browser, a
user can opt to delegate some requests to their own server (as if using a
proxy), thus avoiding some CORS issues.

WebID delegated authentication implies at least three parties:

* **delegator** - the user who initiates the request -
  `https://alice.example.org/card#me`
* **delegatee** - the server to which the request is delegated -
  `https://alice.example.org/`
* **data source server** the server hosting a target resource -
  `https://data-source.org/`

The delegator must indicate that it delegates the server agent (delegatee) to
perform requests on their behalf. The server agent is identified by its own
WebID -- i.e. `https://alice.example.org/agent#i`. The relationship between the
delegator and the delegatee is expressed in RDF using the following predicate:
`http://www.w3.org/ns/auth/acl#delegates`.

This is an example of the triple Alice needs to add to her profile document:

```
<https://alice.example.org/card#me> <http://www.w3.org/ns/auth/acl#delegates> <https://alice.example.org/agent#i> .
```

A typical request takes place according to the following workflow:

```
                <alice#me>                             <agent#i>
[ Delegator ] <------------> [ Delegatee ] <----------------------------> [ Data source server ]
                                              On-Behalf-Of: <alice#me>
```

1. Alice (the delegator) intends to request a `test` resource from the data
  source server. Instead of performing a direct cross-origin request, the client
  (application) sends the request to the delegatee server using a *proxy* URI:

  REQUEST:

  ```
  GET /,proxy?uri=https%3A%2F%2Fdata-source.org%2Ftest HTTP/1.1
  Host: alice.example.org
```

2. The delegatee server then directly requests the resource from the `uri`
  value, making sure to identify itself as `https://alice.example.org/agent#i`  
  by using its own credentials. The delegatee server also adds an extra HTTP
  header to the request, called `On-Behalf-Of`, which is used to indicate that
  the request is performed on behalf of another party.

  REQUEST:

  ```
  GET /test HTTP/1.1
  Host: data-source.org
  On-Behalf-Of: https://alice.example.org/card#me
```

3. At this point, the server `data-source.org` dereferences the WebID found in
  the `On-Behalf-Of` header, and checks if the delegatee's WebID (i.e.
  `https://alice.example.org/agent#i`, the user behind this request) matches one
  of the WebIDs listed in the `http://www.w3.org/ns/auth/acl#delegates` relation
  found in the delegator's profile. If a match is found, the data source server
  may decide to treat the delegator as the real identity to be used for this
  request.

  **IMPORTANT:** The reason why the delegatee must use its own identity (i.e.
  `https://alice.example.org/agent#i`) and credentials, is to avoid confusion as
  to who is actually performing the request. This is particularly important when
  applying access control policies, as we could imagine that certain resources
  may not be accessible for certain agents (delegatees).
