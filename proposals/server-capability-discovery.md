## Server Capability Discovery

(Proposed addition to the [Recommendations for Client App Implementation](https://github.com/solid/solid-spec#recommendations-for-server-implementations) section of the Solid specs.)

There are several server capabilities, features, and API endpoints that clients
need to be able to detect programmatically, when encountering a Solid-compatible
server. This section describes the mechanism for advertising those capabilities
with a combination of a `Link` relation header and a Solid service description
document.

### Motivation and Example Capabilities

Below are some examples of the kind of options that a client needs to be able
to discover about a given Solid-compatible server.

##### Account API Endpoints
Apps that provide Solid user creation functionality, account recovery, and
authentication functionality (such as signin or signout features used by
various authentication protocols) need to be able to know what API endpoints
to send their requests to. For example a Solid user signup app needs to know
the URL to which to send a Create User request. Note that these endpoints cannot
be hardcoded or automatically derived by knowing the top-level domain of a
Solid server, for a number of reasons (such as - the API handlers are located
on separate subdomains or even separate domains, or because the Solid server
is running on an arbitrary subdirectory only, such as
`https://example.com/solid/`).

These account endpoints include:

* New user creation endpoint (for multi-user Solid identity providers, see
  below). Example: `/api/accounts/new`
* Account recovery endpoint (for resetting authentication credentials such as
  generating new WebId-TLS certificates or resetting passwords).
  Example: `/api/accounts/recover`
* Account recovery token validation endpoint.
  Example: `/api/accounts/validateToken`
* Signin endpoint (for authentication mechanisms that require a signin page).
  Example: `/signin`
* Signout endpoint (for ending a user's session with that server).
  Example: `/signout`

##### Multi-user vs Single-user mode
Some clients need to know whether the server is serving as an identity provider
(like `databox.me`), or is running in a standalone/single-user mode (such as
for a personal website).

##### Base (Root) Server URL
Many Solid server implementations allow the owners to specify an arbitrary URL
path on which to run (instead of running on the root `/` resource of a domain).
For example, a personal website might mount a Solid server at
`https://example.com/services/solid/` and not on `https://example.com/`.
This makes guessing the various API endpoints provider by Solid servers almost
impossible. To get around this, the server should specify an absolute URL to
the root Solid server URL on which the various APIs are served.

##### Proxy API endpoint
Some Solid server implementations offer a CORS proxy (to help clients load HTTP
resources from HTTPS pages). To help client libraries use those correctly,
those endpoints should be advertised in the capability document.

### Discovering the Service Capability Document
Clients must discover the location of a given Solid server's capability
description document by doing an HTTP `OPTIONS` request to any resource on
that server and parsing the `service` link header relation. Example:

```http
OPTIONS /example/resource HTTP/1.1
Host: user.example.com

HTTP/1.1 204 No Content
Link: <https://example.com/.well-known/solid>; rel="service", <.acl>; rel="acl", <.meta>; rel="describedBy", <http://www.w3.org/ns/ldp#Container>; rel="type", <http://www.w3.org/ns/ldp#BasicContainer>; rel="type"
```

### Service Description Representation

Currently implemented as JSON (Turtle and JSON-LD implementations on the
roadmap).

Sample request (once the service description URL has been discovered via the
link relation header):

```http
GET /.well-known/solid HTTP/1.1
Host: user.example.com
```

```json
{
    "api": {
        "accounts": {
            "new": "/api/accounts/new",
            "recover": "/api/accounts/recover",
            "signin": "/api/accounts/signin",
            "signout": "/api/accounts/signout",
            "validateToken": "/api/accounts/validateToken"
        }
    },
    "root": "https://user.example.com"
}
```
