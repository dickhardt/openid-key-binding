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

If the RP is running on a device that supports a web browser, it makes an authorization request per [OpenID Connect] 3.1. In addition to the `scope` parameter containing `openid`, and the `response_type` having the value `code`, the scope must also include `dpop`, the request parameters MUST include the `dpop_jkt` parameter having the value of the JWK Thumbprint [RFC7638] of the proof-of-possession public key using the SHA-256 hash function, as defined in [RFC9449] section 10.

The authentication request MUST fail if `dpop` is specified in the `scope` but the request does not include the `dpop_jwt` parameter or `dpop` is not specified in the `scope` and the request includes `dpop_jwt` parameter.

Following is a non-normative example of an authentication request using the authorization code flow:

```text
GET /authorize?
response_type=code
&dpop_jkt=1f2e6338febe335e2cbaa7c7154c3cbdcfd8650f95c5fe7206bb6360e37f4b5a
&scope=openid%20profile%20email%20dpop
&client_id=s6BhdRkqt3
&state=af0ifjsldkj
&nonce=2a50f9ea812f9bb4c8f7
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb HTTP/1.1
Host: server.example.com
```

If the OP does not support key binding for the client, it MUST return the OAuth error **TBC**

## Authentication Request - Device Authorization Flow

If the RP is running on a device that does not support a web browser, it makes an authorization request per [RFC8628] 3.1. In the request, the `scope` parameter MUST contain both `openid` and `dpop`. The request MUST include the `dpop_jkt` parameter having the value of the JWK Thumbprint [RFC7638] of the proof-of-possession public key using the SHA-256 hash function, as defined in [RFC9449] section 10.

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
To do this we set the `nonce` claim of `DPoP` header to the authorization `code`.
The `nonce` claim in the `DPoP` is specified in [RFC9449] section 4.2 and is not the same as the ID Token `nonce` claim defined in [OpenID Connect] 3.1.

Non-normative example:

```text
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
DPoP: eyJhbGciOiJFUzI1NiJ9.eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkV\
 TMjU2IiwiandrIjp7ImNydiI6IlAtMjU2Iiwia3R5IjoiRUMiLCJ4IjoibWptR\
 m1MZm9wVmkwZXRfYTZmZFhUTnJqYVUwR1dlZFN0Y3NfRzU4OEkyMCIsInkiOiJ\
 sMFZwRXlSYzdTdUpfdHFhd2NaQ2VLLXVUOEVPVnF4N3NqTHJGeUJTUllZIn0sI\
 m5vbmNlIjoiU3BseGxPQmVaUVFZYllTNld4U2JJQSJ9.cp8uN3kHAMS9fhGH7T\
 vTSKwH5oNJzAeMhIrgD_HQHGhgt_N1xQHdHiMkn7AMj3UDkwoNOW4Qqak
grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

If a DPoP header is included in the token request to the RP, and the `dpop_jkt` parameter was not included in the authentication request, the OP MUST NOT include the `cnf` claim in the ID Token.

> This prevents an existing deployment using DPoP for access token from having them included in ID Tokens accidentally.

The OP MUST perform all verification steps as described in [RFC9449] section 5.

In addition, the OP MUST also confirm the `code` in the DPoP token `nonce` claim matches the `code` in the token request.

## Token Response

If the token request was successful, the OP MUST return an ID Token containing the `cnf` claim as defined in [7800] set to the jwk of the user's public key and with  `typ` set to `dpop+jwt` in the ID Token's protected header.

Non-normative example of the ID Token payload:

```json
{
    "iss": "https://server.example.com",
    "sub": "24400320",
    "aud": "s6BhdRkqt3",
    "nonce": "n-0S6_WzA2Mj",
    "exp": 1311281970,
    "iat": 1311280970,
    "cnf":
        {
            "jwk": {
                "alg":"ES256",
                "crv": "P-256",
                "kty": "EC",
                "x": "mjmFmLfopVi0et_a6fdXTNrjaU0GWedStcs_G588I20",
                "y": "l0VpEyRc7SuJ_tqawcZCeK-uT8EOVqx7sjLrFyBSRYY"
            }
        }
}
```

# Privacy Considerations

*To be completed.*

# Security Considerations

Proof of possession authentication provides a greater level of security than bearer token authentication. To authenticate with a bearer token, the authentication secret must be sent over the internet to the authenticating party. This presents a risk that the authentication secret be stolen is transit or stolen at the server endpoint and replayed. With proof of possession, the authentication secret, i.e. the private key, never needs to leave the client. This reduces the chance of exposure and allows the client to use additional security mechanisms to protect the private key such as HSMs (Hardware Security Modules) or web browser based SSMs (Software Security Modules).

Public key bound ID Tokens provide a higher level security than bear ID Tokens by using proof of possession rather than bearer authentication. For this reason public key bound ID Tokens MUST NOT be accepted as a form of bearer token authentication.If bearer token authentication is desired, bearer ID Tokens should be used instead.

Within a closed ecosystem, the private key associated with a public key bound ID Token MAY be used to sign and authenticate the content of message. This use REQUIRES that the user's agent enforce secure domain separation via some other protocol between signatures on messages and proof of possession signatures on challenges. Domain separation is need to enforce that the user's signature on a proof of possession challenge message is not replayed in other contexts.

The content of such signed messages MUST NOT be treated as authenticated outside of that ecosystem.

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
