%%%
title = "OpenID Connect Key Binding 1.0 - draft 00"
abbrev = "openid-connect-key-binding"
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

OpenID Connect is a protocol that enables a Relying Party (RP) to delegate authentication and obtain identity claims to an OpenID Connect Provider (OP).

When authenticating with OpenID Connect, an RP provides a nonce in its authentication request. The ID Token signed and returned by the OP contains the nonce and claims about the user. When verifying the ID Token, the RP confirms it contains the nonce, binding the session that made the request to the response.

It is common for an RP to be composed of multiple components. When the RP authenticating component that obtained the ID Token wants to prove to an RP consuming component that it has authenticated a user, it may present the ID Token as a bearer token. However, bearer tokens are vulnerable to theft and replay attacks - if an attacker obtains the ID Token, they can impersonate the authenticated user.

By binding a cryptographic key to the ID Token, the RP authenticating component can prove to RP consuming components not only that a user has been authenticated, but that the RP authenticating component itself was the original recipient of that authentication. This transforms the ID Token from a vulnerable bearer token into a proof-of-possession token that provides stronger security guarantees.

This specification profiles OpenID Connect 1.0, RFC8628 - OAuth 2.0 Device Authorization Grant, and RFC9449 - OAuth 2.0 Demonstrating Proof of Possession (DPoP) to enable cryptographically bound ID Tokens that resist theft and replay attacks while maintaining compatibility with existing OpenID Connect infrastructure.


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

- **OP**: The OpenID Provider as defined in {{OpenID.Core}}.

- **RP**: The Relying Party as defined in {{OpenID.Core}}. 

The parameters **dpop_jkt** and **DPoP** as defined in {{RFC9449}}

## Protocol Profile Overview

This specification profiles how to bind a public key to an ID Token by:

1. adding the `bound_key` scope and `dpop_jkt` parameter to the OpenID Connect Authentication Request
2. receiving the authorization `code` as usual in the Authentication Response
3. adding the `DPoP` header that includes the hash of the `code`, `c_hash`, as a claim in the Token Request to the OP `token_endpoint`
4. adding the `cnf` claim containing the public key to the returned ID Token

```
+------+                              +------+
|      |-- Authentication Request --->|      |
|  RP  |   (1) bound_key & dpop_jkt   |  OP  | 
|      |                              |      | 
|      |<-- Authentication Response --|      |
|      |   (2) authorization code     |      | 
|      |                              |      | 
|      |-- Token Request ------------>|      |
|      |   (3) DPoP header w/ c_hash  |      |
|      |                              |      |
|      |<-- Token Response -----------|      |
|      |   (4) cnf claim containing   |      |
|      |   the public key in ID Token |      | 
+------+                              +------+
```

## OpenID Connect Metadata

The OP's OpenID Connect Metadata Document ({{OpenID.Discovery}})SHOULD include":

- the `bound_key` scope in the `supported_scopes`
- the `dpop_signing_alg_values_supported` property containing a list of supported algorithms as defined in IANA.JOSE.ALGS 


## Authentication Request - Authorization Code Flow

If the RP authenticating component is running on a device that supports a web browser, it makes an authorization request per {{OpenID.Core}} 3.1. In addition to the `scope` parameter containing `openid`, and the `response_type` having the value `code`, the `scope` parameter MUST also include `bound_key`, and the request MUST include the `dpop_jkt` parameter having the value of the JWK Thumbprint {{RFC7638}} of the proof-of-possession public key using the SHA-256 hash function, as defined in {{RFC9449}} section 10.

Following is a non-normative example of an authentication request using the authorization code flow:

```text
GET /authorize?
response_type=code
&dpop_jkt=1f2e6338febe335e2cbaa7c7154c3cbdcfd8650f95c5fe7206bb6360e37f4b5a
&scope=openid%20profile%20email%20bound_key
&client_id=s6BhdRkqt3
&state=af0ifjsldkj
&nonce=2a50f9ea812f9bb4c8f7
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb HTTP/1.1
Host: server.example.com
```

If the OP does not support the `bound_key` scope, it SHOULD ignore it per {{OpenID.Core}} 3.1.2.1.


## Authentication Request - Device Authorization Flow

If the RP authenticating component is running on a device that does not support a web browser, it makes an authorization request per {{RFC8628}} 3.1. In the request, the `scope` parameter MUST contain both `openid` and `bound_key`. The request MUST include the `dpop_jkt` parameter having the value of the JWK Thumbprint {{RFC7638}} of the proof-of-possession public key using the SHA-256 hash function, as defined in {{RFC9449}} section 10.

Following is a non-normative example of an authentication request using the device authorization flow:

```text
TBD

```


If the OP does not support the `bound_key` scope, it SHOULD ignore it per {{OpenID.Core}} 3.1.2.1.


## Authentication Response


If the key provided was not previously bound to the client, the OP SHOULD inform a user and obtain consent that a key binding will be done. 

On successful authentication of, and consent from the user, the OP returns an authorization `code`.

Following is a non-normative example of a response:

```text
TBD
```

## Token Request

To obtain the ID Token, the RP authenticating component:

1. generates a `c_hash` by computing a SHA256 hash of the authorization `code`
2. converts the hash to BASE64URL 
3. generates a `DPoP` header, including the `c_hash` claim in the `DPoP` header JWT. This binds the authorization code to the token request. 

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

If a DPoP header is included in the token request to the OP, and the `dpop_jkt` parameter was not included in the authentication request, the OP MUST NOT include the `cnf` claim in the ID Token.

> This prevents an existing deployment using DPoP for access token from having them included in ID Tokens accidentally.

The OP MUST:
- perform all verification steps as described in {{RFC9449}} section 5.
- calculate the `c_hash` from the authorization `code` just as the RP component did.
- confirm the `c_hash` in the DPoP JWT matches its calculated `c_hash`

## Token Response

If the token request was successful, the OP MUST return an ID Token containing the `cnf` claim as defined in {{RFC7800}} set to the jwk of the user's public key and with  `typ` set to `id_token+cnf` in the ID Token's protected header.

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

## ID Token Proof of Possession

The mechanism for how an RP authenticating component proves to an RP consuming component that it possesses the private keys associated with the `cnf` claim in the ID Token is out of scope of this document.

> If the WG wants to, we can also profile how to use KB to bind a proof of possession to an ID Token for presentation when a proof of possesion is not present.

# Privacy Considerations

An RP authenticating component SHOULD only share an ID Token with a consuming component when such sharing is consistent with the original purpose for which the PII was collected and the scope of consent obtained from the user.

# Security Considerations

## Require Proof of Possesion

An RP consuming component MUST NOT trust an ID Token with a `cnf` claim without a corresponding proof of possession from the RP authenticating component.


## ID Token Reverification

In addition to verifying the signature created by the RP authenticating component to prove possession of the private key associated with the `cnf` claim in the ID Token, an RP consuming component MUST independently verify the signature and validity of the ID Token and that the `aud` claim in the payload is the correct value, and that the `typ` claim in the protected header is `id_token+cnf`.


## Use as Access Token

The RP authenticating component, nor an RP consuming component, MUST NOT use the ID Token as an access token to access resources.

## Unique Key Pair

To prevent token confusion attacks, the RP authenticating component SHOULD bind a unique key pair to its ID Tokens, and not use it for other purposes.

## Using cnf as a User Claim

The `cnf` claim in the ID Token MUST NOT be used as proof a party presenting the ID Token controls the key identified by `cnf`. A  proof of possession is REQUIRED to establish that a party controls the key identified by `cnf`. The `cnf` claim SHOULD only be used to bind a signed object with the other claims in the ID Token.

# IANA Considerations

Media Type Registry
The following entry should be added to the "Media Types" registry for the new JWT type:
Type name: application
Subtype name: id_token+cnf

# References

## Normative References

- **[RFC2119]** Bradner, S. “Key words for use in RFCs to Indicate Requirement Levels,” *RFC 2119*, March 1997.
 - **[RFC7549]** W. Mills, P. Saint-Andre, T. Hardie, “The OAuth 2.0 Authorization Framework: Bearer Token Usage,” RFC 7549, May 2015. Available at <https://datatracker.ietf.org/doc/html/rfc7549>.
 - **[RFC7638]** M. Jones, “JSON Web Key (JWK) Thumbprint,” RFC 7638, September 2015. Available at <https://datatracker.ietf.org/doc/html/rfc7638>.
 - **[RFC7800]** M. Jones, J. Bradley, N. Sakimura, “Proof-of-Possession Key Semantics for JSON Web Tokens (JWTs),” RFC 7800, April 2016. Available at <https://datatracker.ietf.org/doc/html/rfc7800>.
 - **[RFC8628]** W. Denniss, J. Bradley, “OAuth 2.0 Device Authorization Grant,” RFC 8628, June 2019. Available at <https://datatracker.ietf.org/doc/html/rfc8628>.
 - **[RFC9449]** D. Fett, B. Campbell, J. Bradley, “OAuth 2.0 Demonstrating Proof-of-Possession at the Application Layer (DPoP),” RFC 9449, July 2023. Available at <https://datatracker.ietf.org/doc/html/rfc9449>.


- **OpenID.Core** – “OpenID Connect Core 1.0 incorporating errata set 2,” available at <https://openid.net/specs/openid-connect-core-1_0.html>.
- **OpenID.Discovery** – “OpenID Connect Discovery 1.0,” available at <https://openid.net/specs/openid-connect-discovery-1_0.html>.

## Informative References

- **IANA JSON Web Token Claims Registry**, available at <https://www.iana.org/assignments/jwt/jwt.xhtml>.
- **IANA OAuth Parameters**, available at <https://www.iana.org/assignments/oauth-parameters/oauth-parameters.xhtml#client-metadata>.

{backmatter}

# Acknowledgements


The authors would like to thank early feedback provided by Filip Skokan, George Fletcher, Jacob Ideskog, and Kosuke Koiwai.

# Notices

*To be completed if adopted.*


# Document History

   [[ To be removed from the final specification ]]

   -00

   initial draft
