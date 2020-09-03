> This Repo is a Legacy Repo and no longer actively maintained. Please visit [Solid's official website](https://solidproject.org/) instead.

### WebID-RSA

WebID-RSA is somewhat similar to WebID-TLS, in that a public RSA key is
published in the WebID profile, and the user will sign a token with the
corresponding private key that matches the public key in the profile.

The client receives a secure token from the server, which it signs and then
sends back to the server. The implementation of WebID-RSA is similar to [Digest
access authentication](https://tools.ietf.org/html/rfc2617) in HTTP, in that it
reuses the same headers.

Here is a step by step example that covers the authentication handshake.

First, the client attempts to access a protected resource at
`https://example.org/data/`.

REQUEST:

```
GET /data/ HTTP/1.1
Host: example.org
```

RESPONSE:

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: WebID-RSA source="example.org", nonce="securestring"
```

Next, the client sets the username value to the user's WebID and signs the
`SHA1` hash of the concatenated value of **source + username + nonce** before
resending the request. The signature must use the `PKCS1v15` standard and it
must be `base64` encoded.

It is important that clients return the proper source value they received from
the server, in order to avoid man-in-the-middle attacks. Also note that the
server must send its own URI (**source**) together with the token, otherwise a
[MitM](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) can forward the
claim to the client; the server will also expect that clients return the same
server URI.

REQUEST:

```
GET /data/ HTTP/1.1
Host: example.org
Authorization: WebID-RSA source="example.org",
                         username="https://alice.example.org/card#me",
                         nonce="securestring",
                         sig="base64(sig(SHA1(SourceUsernameNonce)))"
```

RESPONSE:

```
HTTP/1.1 200 OK
```

One important advantage of WebID-RSA over WebID-TLS is that keys can be
generated on the fly to sign and encrypt data. The way client certificate
management is currently implemented in browsers, it does not offer the means to
access keys inside certificates, for purposes other than authentication.

***Proposed improvement:*** (not implemented yet) Instead of sending the WebID
during the response, the client could directly send the URI of the public key
that is need in order to verify the claim. For instance, Alice could list public
keys in her own profile, using fragment identifiers (e.g. `<#key1>`):

```
....
<#me> cert:key <#key1>, <#key2> .

<#key1> a cert:RSAPublicKey;
        cert:modulus "00cb24ed85d64d794b..."^^xsd:hexBinary;
        cert:exponent 65537  .
```

The client would then send the following response:

```
GET /data/ HTTP/1.1
Host: example.org
Authorization: WebID-RSA keyuri="https://alice.example.org/card#key1",
                         nonce="securestring",
                         sig="signatureOverUsernamePlusNonce"
```

The server would then be able to immediately identify and link the key that was
used to sign the response to the user that owns it.
