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

TODO Abstract

--- note_Current_Implementations

This note is to be removed before publishing as an RFC.

Let's Encrypt's [Boulder](https://github.com/letsencrypt/boulder) ACME Server software fully implements this draft. Let's Encrypt has not yet configured profiles to be advertised in their [Production](https://acme-v02.api.letsencrypt.org/directory) and [Staging](https://acme-staging-v02.api.letsencrypt.org/directory) environments.  The [Pebble](https://githbu.com/letsencrypt/pebble) ACME Server testbed also implements this draft.

--- middle

# Introduction

TODO Introduction

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Extensions to the Directory Resource

An ACME Server which wishes to allow Clients to select profiles MUST include a new field, `profiles`, in the `meta` section of its Directory object. This field is itself a mapping of profile names to human-readable descriptions of those profiles.

TODO:

- Descriptions may be prose or links to documentation pages
- Example

# Extensions to the Order Resource

In order to convey information about the profile associated with an Order, a new field is added to the Order object:

`profile` (string, optional): A string uniquely identifying the profile which will be used to affect issuance of the certificate requested by this Order.

TODO:

- Clients must only request profiles which are advertised in the directory
- Servers must reject any request which specifies an unrecognized profile
- Servers may assign a profile to requests which do not specify one
- Request and response examples

# Security Considerations

The extensions to the ACME protocol described in this document build upon the Security Considerations and threat model defined in {{Section 10.1 of RFC8555}}.

TODO:

- ACME Servers must ensure that all profiles they offer abide by the requirements which govern their PKI, such as their CPS
- Only allowing a single profile to be requested, i.e. not offering a a buffet of options that can be mixed and matched, greatly simplifies this compliance
- Transporting profile information via ACME rather than via CSR reduces incidents

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

--- back

# Acknowledgments
{:numbered="false"}

My thanks to Phil Porada for spearheading the implementation of this protocol in the Boulder software. Thanks also to Jacob Hoffman-Andrews, Samantha Frank, James Kasten, and Jason Baker for discussions and brainstorming on what this protocol should look like.
