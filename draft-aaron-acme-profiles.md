---
title: "Automated Certificate Management Environment (ACME) Profiles Extension"
abbrev: "ACME Profiles"
category: info

docname: draft-aaron-acme-profiles-latest
submissiontype: IETF
consensus: true
v: 3
area: "Security"
workgroup: "Automated Certificate Management Environment"
venue:
  group: "Automated Certificate Management Environment"
  type: "Working Group"
  mail: "acme@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/acme/"
  github: "aarongable/draft-acme-profiles"
  latest: "https://aarongable.github.io/draft-acme-profiles/draft-aaron-acme-profiles.html"

author:
 -
    fullname: "Aaron Gable"
    organization: Internet Security Research Group
    email: "aaron@letsencrypt.org"

normative:
  RFC8555:

informative:

--- abstract

This document defines how an ACME Server may offer a selection of different certificate profiles to ACME Clients, and how those clients may indicate which profile they want.

--- note_Current_Implementations

This note is to be removed before publishing as an RFC.

Let's Encrypt's [Boulder](https://github.com/letsencrypt/boulder) ACME Server software fully implements this draft. Let's Encrypt has not yet configured profiles to be advertised in their [Production](https://acme-v02.api.letsencrypt.org/directory) and [Staging](https://acme-staging-v02.api.letsencrypt.org/directory) environments.  The [Pebble](https://github.com/letsencrypt/pebble) ACME Server testbed also implements this draft.

--- middle

# Introduction

Throughout the history of the WebPKI, Certificate Authorities have used "profiles" to describe the kinds of certificates they issue. For example, an "S/MIME" profile might indicate that the resulting certificate will contain the `id-kp-emailProtection` Extended Key Usage and use a Certificate Policy OID to assert compliance with the CA/Browser Forum S/MIME Baseline Requirements; or a "Constrained Sub-CA" profile might indicate the inclusion of the Basic Constraints extension, the keyCertSign Key Usage, and a Name Constraints extension. Subscribers generally select a profile as part of their certificate ordering process or negotiations with the CA, depending on their needs (and sometimes their budget).

The ACME protocol only allows clients to customize their certificate in two ways: the newOrder request allows selection of the identifiers (generally Subject Alternative Names) and validity period; and the finalize request contains a CSR in which the client can theoretically include any combination of fields and extensions that they desire. But requesting certificate features via the CSR is onerous, error-prone, and dangerous. Numerous compliance incidents across the WebPKI have been caused by CAs trusting values included in a CSR and copying them directly into the issued certificate. It requires clients to know how to construct a valid CSR, provides no mechanism for servers to advertise what CSR fields they're willing to accept, and means that CAs have to evaluate on a case-by-case basis which combinations of requested features they're willing to issue.

This document provides a mechanism for ACME Servers to advertise what certificate profiles they offer, and for ACME Clients to select a profile when creating a new Order. This allows site operators to make informed decisions about the kind of certificate they request, without placing an undue burden on ACME Clients or Servers to transmit such information in the form of a CSR. It also encourages the evolution of the WebPKI by allowing CAs to more easily offer new and improved profiles while maintaining backwards compatibility for old subscribers.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Extensions to the Directory Resource

An ACME Server which wishes to allow Clients to select profiles MUST include a new field, `profiles`, in the `meta` field of its Directory object.

`profiles` (optional, object):  A map of profile names to human-readable descriptions of those profiles.

The contents of these human-readable descriptions are up to the CA; for example, they might be prose descriptions of the properties of the profile, or the might be URLs pointing at a documentation site. ACME Clients SHOULD present these profile names and descriptions to their operator during initial setup and at appropriate times thereafter.

~~~ json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "newNonce": "https://example.com/acme/new-nonce",
  "newAccount": "https://example.com/acme/new-account",
  "newOrder": "https://example.com/acme/new-order",
  "newAuthz": "https://example.com/acme/new-authz",
  "revokeCert": "https://example.com/acme/revoke-cert",
  "keyChange": "https://example.com/acme/key-change",
  "meta": {
    "termsOfService": "https://example.com/acme/terms/2021-10-05",
    "website": "https://www.example.com/",
    "caaIdentities": ["example.com"],
    "externalAccountRequired": false,
    "profiles": {
      "profile1": "https://example.com/docs/profiles#profile1",
      "profile2": "https://example.com/docs/profiles#profile2",
    }
  }
}
~~~

# Extensions to the Order Resource

In order to convey information about the profile associated with an Order, a new field is added to the Order object:

`profile` (string, optional): A string uniquely identifying the profile which will be used to affect issuance of the certificate requested by this Order.

To select a profile, the client includes the desired profile name in the `profile` field of the Order object they supply to the newOrder request. The client MUST NOT request a profile name that is not advertised in the server's Directory metadata object.

~~~ text
POST /acme/new-order HTTP/1.1
Host: example.com
Content-Type: application/jose+json

{
  "protected": base64url({
    "alg": "ES256",
    "kid": "https://example.com/acme/acct/evOfKhNU60wg",
    "nonce": "5XJ1L3lEkMG7tR6pA00clA",
    "url": "https://example.com/acme/new-order"
  }),
  "payload": base64url({
    "profile": "profile1"
    "identifiers": [{"type": "dns", "value": "example.org"}],
  }),
  "signature": "H6ZXtGjTZyUnPeKn...wEA4TklBdh3e454g"
}
~~~

If the server receives a newOrder request specifying a profile that it is not advertising, or specifying a profile which is incompatible with the rest of the contents of the request (e.g. a "tls-server-auth" profile alongside an identifier of type "email"), it MUST reject the request with a problem document of type "invalidProfile" (see Section 6.3).

If the server is advertizing profiles and receives a newOrder request which does not identify a specific profile, it is RECOMMENDED that the server select a profile and associate it with the new Order object.

~~~ text
   {
     "status": "valid",
     "expires": "2025-01-01T12:00:00Z",
     "profile": "profile1",
     "identifiers": [{"type": "dns", "value": "example.org"}],
     "authorizations": ["https://example.com/acme/authz/PAniVnsZcis"],
     "finalize": "https://example.com/acme/order/TOlocE8rfgo/finalize",
   }
~~~

If a server receives a request to finalize an Order whose profile the CA is no longer willing to issue under, it MUST respond with a problem document of type "invalidProfile". The server SHOULD attempt to avoid this situation, e.g. by ensuring that all Orders for a profile have expired before it stops issuing under that profile.

# Security Considerations

The extensions to the ACME protocol described in this document build upon the Security Considerations and threat model defined in {{Section 10.1 of RFC8555}}. It does not change the account management or identifier validation flows, so the security considerations are largely unchanged.

The one exception is in regards to CA Policy Considerations. RFC 8555 did not address how a server should determine whether it is willing to issue the certificate as requested by the finalize CSR. This document greatly simplifies this determination by making the contents of the CSR (beyond the Subject Alternative Names and Subject Public Key) irrelevant to the decision-making process.

Additionally, by moving profile selection out of the finalize CSR and into the Order resource itself, this document reduces the need for ACME Clients and Servers to parse and process x509 ASN.1. This increases the security and stability of the WebPKI as a whole by reducing the incidence of parsing errors and the likelihood of values being copied directly from the CSR into the resulting certificate.

# IANA Considerations

## ACME Directory Metadata Fields

IANA will add the following entry to the "ACME Directory Metadata Fields" registry within the "Automated Certificate Management Environment (ACME) Protocol" registry group at <https://www.iana.org/assignments/acme>:

Field Name  | Field Type | Reference
------------|------------|-----------
profiles    | object     | This document

## ACME Order Object Fields

IANA will add the following entry to the "ACME Order Object Fields" registry within the "Automated Certificate Management Environment (ACME) Protocol" registry group at <https://www.iana.org/assignments/acme>:

Field Name | Field Type | Configurable | Reference
-----------|------------|--------------|-----------
profile    | string     | true         | This document

## ACME Error Types

IANA will add the following entry to the "ACME Error Types" registry within the "Automated Certificate Management Environment (ACME) Protocol" registry group at <https://www.iana.org/assignments/acme>:

Type           | Description | Reference
---------------|-------------|-----------
invalidProfile | The request specified a profile which this server does not support | This document

--- back

# Acknowledgments
{:numbered="false"}

My thanks to Phil Porada for spearheading the implementation of this protocol in the Boulder software. Thanks also to Jacob Hoffman-Andrews, Samantha Frank, James Kasten, and Jason Baker for discussions and brainstorming on what this protocol should look like.
