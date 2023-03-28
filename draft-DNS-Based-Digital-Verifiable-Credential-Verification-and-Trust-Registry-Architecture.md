---
title: "DNS Based Digital Verifiable Credential Verification and Trust Registry Architecture"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-DNS-Based-Digital-Verifiable-Credential-Verification-and-Trust-Registry-Architecture-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 0

keyword:

- trust registry
- distributed ledger
- did
- verifiable credential
- dns

venue:
  github: "CIRALabs/DNS-Based-VCs-and-Trust-Registries-ID"
  latest: "https://CIRALabs.github.io/DNS-Based-VCs-and-Trust-Registries-ID/draft-DNS-Based-Digital-Verifiable-Credential-Verification-and-Trust-Registry-Architecture.html"

author:
 -
    fullname: J.Latour
    organization: CIRA
 -
    fullname: J.Carter
    organization: CIRA
 -
    fullname: M.Glaude
    organization: NorthernBlock

normative:

informative:

--- abstract

This memo describes an architecture for digital credential verification and validation using Decentralized Identifiers (DIDs), distributed ledgers, trust registries, and the DNS.

--- middle

# Introduction

This memo describes an architecture for digital credential verification using distributed ledgers, the DNS, and trust registries.

The use case involves an individual or an organization receiving a verifiable credential [AnonCreds](https://hyperledger.github.io/anoncreds-spec/) [W3C](https://www.w3.org/TR/vc-data-model/) from an issuer and storing it in their digital wallet. When the individual needs to provide proof of identity or other claims, they present the verifiable credential to a verifier in the form of a verifiable claim which normally includes a digital signature. The verifier then performs several steps to verify the authenticity of the credential, including extracting the issuer's DID from the credential, resolving it on a distributed ledger to obtain the issuer's DID document, verifying the signature of the credential using the public key in the issuer's DID document, verifying the issuer's domain name and public key through DNS queries using URI and TLSA records, and finally verifying the issuer through a trust registry rooted in the DNS using URI and TLSA records, while ensuring  all these DNS records are properly signed and validated with DNSSEC.

This process allows for secure and decentralized verification of digital credentials in a manner that is transparent and auditable. It provides individuals with greater control over their personal data while also enabling organizations to verify identity claims in a trustworthy manner.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

- Issuer: TODO - add definition
- Verifier: TODO - add definition
- Trust Registry: TODO - add definition

# Goal

The memo aims to improve global interoperability between different decentralized digital identity ecosystems by ensuring that issuers and digital credentials have unique identifiers globally. The memo also aims to demonstrate that a Trust Registry can enable global interoperability by providing a layer of digital trust in the use of digital credentials and demonstrate that Trust Registries can enable a more efficient and trustworthy credential verification process. By leveraging the publicly resolvable and widely supported DNS/DNSSEC infrastructure, verifiers can easily validate not only the integrity of the credential they are presented with but also quickly associate the issuer of that credential with a domain name and organization, as well as their authority and trustworthiness by confirming their membership in a Trust Registry.

# Mapping a DID to the DNS

The W3C DID Core spec supports multiple ways of associating a DID to a domain.

alsoKnownAs: The assertion that two or more DIDs (or other types of URI, such as a domain name) refer to the same DID subject can be made using the [alsoKnownAs](https://www.w3.org/TR/did-core/#also-known-as) property.

Services: Alternatively, [services](https://www.w3.org/TR/did-core/#services) are used in DID documents to express ways of communicating with the DID subject or associated entities. In this case we are referring specifically to the "LinkedDomains" service type.

However, this association stemming only from the DID is unidirectional. By leveraging URI records as outlined in [DID in the DNS](https://datatracker.ietf.org/doc/html/draft-mayrhofer-did-dns-05#section-2), we can create a bidirectional relationship, allowing a domain to publish their associated DIDs in the DNS.

**Ex: _did.example-issuer.ca IN URI 1 0 “did:sov:123456”**

This relationship enhances security, as an entity would require control over both the DID and the domain’s DNS server to create this bidirectional association, reducing the likelihood of malicious impersonation.

The ability for an organization to publish a list of their DIDs on the DNS is also beneficial as it establishes a link between the DNS, which is ubiquitously supported, and the distributed ledger the DIDs resides on which may not have the same degree of access or support, enhancing discoverability.

## URI record scoping

- The records MUST be scoped by setting the global underscore name of the URI RRset to ‘_did’ (0x5F 0x64 0x69 0x64).

## Issuer Handles

An issuer may have multiple sub entities issuing credentials on their behalf, such as the different faculties in a university issuing diplomas. Each of these entities will need to be registered separately in a trust registry and will likely have one or more DIDs of their own. For this reason, the introduction of an issuer handle, represented as a subdomain in the resource record name, provides a simple way to facilitate the distinction of DIDs, their public keys, and credentials they issue in their relationship to the issuer.

**Ex: _did.diplomas.university-issuer.ca IN URI 1 0 “did:sov:XXXXXXX”**

**Ex: _did.certificates. university -issuer.ca IN URI 1 0 “did:sov:YYYYYYY”**

# DID Public Keys in the DNS

The DID to DNS mapping illustrated in section 4 provides a way of showing the association between a DID and a domain, but no way of verifying that relationship. By hosting the public keys of that DID in its related domain’s zone, we can provide a cryptographic linkage to bolster this relationship while also providing access to the DID’s public keys outside of the distributed ledger where it resides, facilitating interoperability.

[TLSA records](https://www.rfc-editor.org/rfc/rfc6698) provide a simple way of hosting cryptographic information in the DNS.

## TLSA Record Scoping, Selector Field

When public keys related to DIDs are published in the DNS as TLSA records:

- The records MUST be scoped by setting the global underscore name of the TLSA RRset to ‘_did’ (0x5F 0x64 0x69 0x64).
- The Selector Field of the TLSA record must be set to 1, SubjectPublicKeyInfo: DER-encoded binary structure as defined in [RFC5280](https://www.rfc-editor.org/rfc/rfc5280).

## Issuer Handles

As mentioned in section 4.2, an Issuer may have multiple sub entities issuing credentials on their behalf, likely with their own set/s of keypairs. Because these keypairs will need to be registered in a trust registry, and represented in the DNS as TLSA records, the use of an Issuer Handle as outlined in section 4.2 will facilitate the distinction of the different public keys in their relation to the issuer.

**Ex: _did.diplomas.university-issuer.ca IN TLSA 3 0 0 “4e18ac22c00fb9 b96270a7b2”**

**Ex: _did.certificates. university -issuer.ca IN TLSA 3 0 0 “4e18ac22c00fb9 b96270a7b3”**

## Instances of Multiple Key Pairs

Depending on the needs of the Issuer, it is possible they may use multiple keypairs associated with a single DID to sign and issue credentials. In this case a mechanism to differentiate which verificationMethod the public key is related to will need to be added to the name of the TLSA RRset.

A simple solution would be to create a standardized naming convention by expanding the RRset name using the fragment fragment of the target verificationMethod.

**Ex: _did.key-1.example-issuer.ca IN TLSA 3 0 0 ‘someHexKey’**

**Ex: _did.key-2.example-issuer.ca in TLSA 3 0 0 ‘anotherHexKey’**

## Benefits of Public Keys in the DNS

Hosting the public keys in TLSA records provides a stronger mechanism for the verifier to verify the issuer with, as they are able to perform a cryptographic challenge against the DID using the corresponding TLSA records, or against the domain using the corresponding [verificationMethod](https://www.w3.org/TR/did-core/#verification-methods) in the DID document. The accessibility of the public keys is also a benefit, as the verifier only needs to resolve the DID document on the distributed ledger and can perform the remainder of the verification process using data available in the DNS, potentially limiting the burden of having to interoperate with a multitude of different distributed ledger technologies and transactions for key access.

# Digital Credential Verification using DIDs and the DNS

By leveraging the records and relationships outlined above, the verifier can verify a digital credential claim by:

- Looking up the credential’s Issuer’s DID on a distributed ledger to resolve a DID document.
- Extracting the Issuer’s domain from that DID document.
- Querying that domain for a _did URI record to confirm the Issuer’s domain is also claiming ownership over that DID.
- Performing a verification of the credential’s signature/proof using the relevant verificationMethod in the DID document.
- Querying that domain for a _did TLSA record to confirm the public key expressed by the verificationMethod is also expressed by the Issuer’s domain.

Through this process, the Verifier has not only cryptographically verified the credential they are being presented with, but also associated the issuing DID to a publicly resolvable domain, confirming it’s validity both semantically and cryptographically.

# Role of DNSSEC for Assurance and Revocation

It is a MUST that all the participants in this digital identity ecosystem to enable DNSSEC signing for all the DNS instance they operate align with DNSSEC validation. See [RFC 9364 - DNS Security Extensions (DNSSEC)](https://datatracker.ietf.org/doc/html/rfc9364)

DNSSEC provides cryptographic assurance that the DNS records returned in response to a query are authentic and have not been tampered with. This assurance within the context of the _did URI and _did TLSA records provides another mechanism to ensure the integrity of the DID and its public keys outside of the distributed ledger it resides on directly from the domain of its owner.

Within this use-case, DNSSEC also provides revocation checks for both DIDs and public keys.. In particular, a DNS query for a specific _did URI record or _did TLSA record can return an [NXDOMAIN](https://www.rfc-editor.org/rfc/rfc8020) response if the DID or public key has been revoked.  This approach can simplify the process of verifying the validity of DIDs and public keys by reducing the need for complex revocation mechanisms.

# The Role of Trust Registries in Bidirectional Credential Verification

A trust registry is a decentralized system that enables the verification of the authenticity and trustworthiness of issuers of digital credentials. Trust registries can be implemented using distributed ledger technology, such as the DNS, to provide a transparent and auditable record of issuer information.

When an entity is presented with a verifiable claim, there are three things they will want to ensure:

1. That a claim hasn’t been altered/falsified at any point in time, via cryptographic verifiability and Verifiable Data Registries (VDRs).
2. That a claim has accurate representation via authentication, DID Discovery & mapping within DNS as described above.
3. That a claim has authority, that is does the Issuer have authority in its issuance of credentials, Via the use of Trust Registries (trust lists)

Trust Registry enables the verification of the authority of an issuer and its claim.  The role of a Trust Registry within the context of this document is to confirm the authenticity and trustworthiness of the Issuer to the Verifier after they have validated the digital credential using the mechanisms described previously. This involves the Trust Registry taking on the role of a trust anchor, providing input for the Verifier’s ultimate trust decision regarding the credential they are being presented with. The assumption is made that the Trust Registry would be operated under the authority of an institution or organization such that their claims and input to the trust decision would be considered significant or definitive. An example of such an organization would be a government entity in relation to the issuance of a Driver’s licenses.

It is important to note that the DNS based Trust Registry mechanism described in this section is not meant to operate in place of an alternative implementation but provide an easy to implement and use mechanism to extend such a solution.

This section also does not describe the process of the Trust Registry’s verification of an Issuer, or the process of how an Issuer would become accredited by or join a Trust Registry.

## Issuer's Membership Claim in a Trust Registry

Once the Verifier has successfully completed the credential verification process outlined in section 6, they have definitive proof that the credential they are being presented with was issued by the claimed issuer, and that issuer can be resolved to an organization’s or entity’s domain. However, this process does not provide definitive proof the Issuer is to be trusted or has the authority to issue such a credential. The Issuer, through use of URI records and the _trustregistry label, can assert the claim that they are a member of a Trust Registry.

**Ex: _trustregistry.example-issuer.ca IN URI 0 1 “example-trustregistry.ca”**

This record indicates the Verifier can then query the “example-trustregistry.ca” for further URI and TLSA records proving “example-issuer.ca”s membership.

### URI Record Name Scoping

When Trust Registry membership claims are published in the DNS

- The records MUST be scoped by setting the global (highest-level) underscore name of the URI RRset to ‘_trustregistry’ (0x5F 0x74 0x72 0x75 0x73 0x74 0x72 0x65 0x67 0x69 0x73 0x74 0x72 0x79)

## Trust Registry Membership Proof

The Trust Registry can assert an Issuer’s membership using TLSA records in a similar fashion to the methods outlined by section 5.1.

**Ex: _example-issuer.ca._trustregistration.example-trustregistry.ca in TLSA 3 0 0 “SomeHexKey”**

Note that the first component of the URI is the Issuer’s domain, followed by the _trustregistration label. This combination indicates that the domain expressed is registered by this trust registry as per its governance model, and this is their public key. This association created by the TLSA record effectively has created a chain of trust, beginning at the DID’s verificationMethod, continuing to the Issuer’s domain, and finally resolving at the Trust Registry.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
