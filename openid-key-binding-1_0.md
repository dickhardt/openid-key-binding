%%%
title = "OpenID Key Binding 1.0 - draft 00"
abbrev = "openid-key-binding"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid", "lifecycle"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-key-binding-1_0"
status = "standard"

[[author]]
initials="D."
surname="Hardt"
fullname="Dick Hardt"
organization="Hellō"
    [author.address]
    email = "dick.hardt@gmail.com"

[[author]]  
initials="E."
surname="Heilman"
fullname="Ethan Heilman"
organization="Cloudflare"
    [author.address]
    email = "ethan.r.heilman@gmail.com"

%%%

.# Abstract

OpenID Key Binding specifies how to bind a public key to an OpenID Connect ID Token using mechanisms defined in RFC 9449, OAuth 2.0 Demonstrating Proof of Possession (DPoP).

{mainmatter}

# Introduction

Some applications use proof of possession of a private key for authentication. OpenID Connect provides a protocol to delegate authentication and obtain identity claims to another service. This document specifies an extension to OpenID Connect to bind a public key to an OpenID Connect ID Token by profiling OpenID Connect 1.0, RFC8628 - OAuth 2.0 Device Authorization Grant, and RFC9449 - OAuth 2.0 Demonstrating Proof of Possession (DPoP).


## Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

In the .txt version of this specification,
values are quoted to indicate that they are to be taken literally.
When using these values in protocol messages,
the quotes MUST NOT be used as part of the value.
In the HTML version of this specification,
values to be taken literally are indicated by
the use of *this fixed-width font*.

## Terminology

This specification uses the following terms:

- **OP**: The OpenID Provider as defined in [OpenID Connect].

- **RP**: The Relying Party as defined in [OpenID Connect]. 

The parameters **dpop_jkt** and **DPoP** as defined in [RFC9449]

## Protocol Profile Overview

This specification profiles how to bind a public key to an ID Token by:

1. adding the `dpop_jkt` parameter to the OpenID Connect Authentication Request
2. receiving the authorization `code` as usual in the Authentication Response
3. adding the `DPoP` header that includes the `code` as a claim to the Token Request to the OP `token_endpoint`
4. adding the `cnf` claim containing the public key to the returned ID Token

```
+------+                              +------+
|      |-- Authentication Request --->|      |
|  RP  |   (1) dpop_jkt parameter     |  OP  | 
|      |                              |      | 
|      |<-- Authentication Response --|      |
|      |   (2) authorization code     |      | 
|      |                              |      | 
|      |-- Token Request ------------>|      |
|      |   (3) DPoP header            |      |
|      |                              |      |
|      |<-- Token Response -----------|      |
|      |   (4) cnf claim containing   |      |
|      |   the public key in ID Token |      | 
+------+                              +------+
```

## Authentication Request - Authorization Code Flow

If the RP is running on a device that supports a web browser, it makes an authorization request per [OpenID Connect] 3.1. In addition to the `scope` parameter containing `openid`, and the `response_type` having the value `code`, the request parameters MUST include the `dpop_jkt` parameter having the value of the JWK Thumbprint [RFC7638] of the proof-of-possession public key using the SHA-256 hash function, as defined in [RFC9449] section 10.

> require `nonce`?

Following is a non-normative example of an authentication request using the authorization code flow:

```text

```

If the OP does not support key binding for the client, it MUST return the OAuth error **TBC**

## Authentication Request - Device Authorization Flow

If the RP is running on a device that does not support a web browser, it makes an authorization request per [RFC8628] 3.1. In the request, the `scope` parameter MUST contain `openid` and the request MUST include the `dpop_jkt` parameter having the value of the JWK Thumbprint [RFC7638] of the proof-of-possession public key using the SHA-256 hash function, as defined in [RFC9449] section 10.

> require `nonce`?


Following is a non-normative example of an authentication request using the device authorization flow:

```text

```

If the key provided was not previously bound to the client, the OP SHOULD inform a user and obtain consent that a key binding will be done. 

If the OP does not support key binding for the client, it MUST return the OAuth error **TBC**

## Authentication Response

On successful authentication of, and consent from the user, the OP returns an authorization `code`.

Following is a non-normative example of response:

```text

```

## Token Request

Generate a `DPoP` header that includes the authorization `code`, that then binds the `DPoP` header to the `code` in the body of the token request.

Non-normative example:

```text

```

If a DPoP header is included in the token request to the RP, and the `dpop_jkt` parameter was not included in the authentication request, the OP MUST NOT include the `cnf` claim in the ID Token.

> This prevents an existing deployment using DPoP for access token from having them included in ID Tokens accidently.

The OP MUST perform all verification steps as described in [RFC9449] section 5.

In addition, the OP MUST also confirm the `code` in the DPoP token matches the `code` in the token request.


## Token Response 

If the token request was successful, the OP MUST return an ID Token containing the `cnf` claim as defined in [7800] set to the `

> specify token `typ`?

Non-normative example of the ID Token payload:

```json

```


# Privacy Considerations

*To be completed.*

# Security Considerations

*To be completed.*

# IANA Considerations

No new registrations.

# References

## Normative References

- **[RFC2119]** Bradner, S. “Key words for use in RFCs to Indicate Requirement Levels,” *RFC 2119*, March 1997.
- **[RFC7549]**
- **[RFC7800]**
- **[RFC9449]** 

- **OpenID Connect Core 1.0** – “OpenID Connect Core 1.0 incorporating errata set 1,” available at <https://openid.net/specs/openid-connect-core-1_0.html>.

## Informative References

- **IANA JSON Web Token Claims Registry**, available at <https://www.iana.org/assignments/jwt/jwt.xhtml>.
- **IANA OAuth Parameters**, available at <https://www.iana.org/assignments/oauth-parameters/oauth-parameters.xhtml#client-metadata>.

{backmatter}

# Acknowledgements

*To be completed.*

# Notices

*To be completed if adopted.*


# Document History

   [[ To be removed from the final specification ]]

   -00

   initial draft
