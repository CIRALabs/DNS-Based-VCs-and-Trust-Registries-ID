---
title: "Leveraging DNS in Digital Trust: Credential Exchanges and Trust Registries"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-latour-dns-and-digital-trust-latest
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
    fullname: Jesse Carter
    organization: CIRA
    email: jesse.carter@cira.ca
 -
    fullname: Jacques Latour
    organization: CIRA
    email: jacques.latour@cira.ca
 -
    fullname: Mathieu Glaude
    organization: NorthernBlock
    email: mathieu@northernblock.io

normative:

informative:

--- abstract

This memo describes an architecture for digital credential verification and validation using Decentralized Identifiers (DIDs), distributed ledgers, trust registries, and the DNS. This architecture provides a verifier with a simple process by which to cryptographically verify the credential they are being presented with, verify and resolve the issuer of that credential to a domain, and verify that issuer's membership in a trust registry.

--- middle

# Introduction

With the increasing adoption and deployment of digital credentials around the world, as well as the numerous different standards and implementations surrounding them, there is a strong likelihood the digital credential ecosystem will become fragmented. There all already 150+ DID Methods listed in the [DID Specification Registries](https://www.w3.org/TR/did-spec-registries/#did-methods), meaning that implementers of digital credentialing solutions would have to ensure they can support the resolution of the right DID Methods that are being used in their interactions. This will present a significant burden to implementers across different nations and organizations, creating large barriers to interoperability.

This memo aims to improve global interoperability between different decentralized digital identity ecosystems by ensuring that public DID owners (i.e. credential issuers and sometimes verifiers) have unique and accessible global identifiers. The memo also aims to demonstrate how trust registries can enable global interoperability by providing a layer of digital trust in the use of digital credentials, demonstrating that trust registries can facilitate a more efficient and trustworthy credential verification process. By leveraging the publicly resolvable and widely supported DNS/DNSSEC infrastructure, entities looking to make a trust decision can easily validate not only the integrity of the credential they are presented with, but also quickly associate the another entity in question with a domain name and organization, as well as their authority and trustworthiness by confirming their membership in a trust registry. We will explore how this implementation can present a more decentralized approach to making trust decisions, without having to integrate directly to all trust registries, but instead letting entities involved in private transactions leverage existing internet infrastructure to facilitate their own trust decisions.

We will focus this memo around a use case involving an individual or an organization receiving a verifiable credential [AnonCreds](https://hyperledger.github.io/anoncreds-spec/) [W3C](https://www.w3.org/TR/vc-data-model/) from an issuer and storing it in their digital wallet. When the individual needs to provide proof of identity or other claims, they present the verifiable credential to a verifier in the form of a verifiable claim which normally includes a digital signature. The verifier then performs several steps to verify the authenticity of the credential, including extracting the issuer's DID from the credential, resolving it on a distributed ledger (Indy ledger) to obtain the issuer's DID document, verifying the signature of the credential using the public key in the issuer's DID document, verifying the issuer's domain name and public key through DNS queries using URI and TLSA records, and finally verifying the issuer through a trust registry grounded in the DNS using URI and TLSA records, while ensuring all these DNS records are properly signed and validated with DNSSEC.

This process allows for the secure and decentralized verification of digital credentials in a manner that is transparent and auditable, while also existing alongside and independent of the many different decentralized identity ecosystems and implementations by grounding itself in the DNS.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

- Issuer: The source of credentials—every verifiable credential has an issuer. Issuers can include organizations such as government agencies (passports, verified person), financial institutions (credit cards), universities (degrees), corporations (employment IDs), churches (awards), etc. Individuals can issue themselves self-attested credentials - and depending on the governance framework of a digital credentialing ecosystem, individuals could issue credentials to others.
- Holder: The recipient of digital credentials. The Holder stores their credentials inside a digital wallet and uses agent technology to interact with other entities. A Holder can be a person, an organization or a machine.
- Verifier: Can be anyone seeking trust assurance of some kind about the holder of a credential. Verifiers request the credentials they need and then follow their own policy to verify their authenticity and validity. For example, a TSA agent at an airport will look for specific features of a passport or driver’s license to see if it is valid, then check to ensure it is not expired.
- Digital Wallet/Agent: A digital wallet, in the context of digital identity, is a secure platform or application that stores and manages an individual's personal identification and authentication credentials, such as government-issued IDs, passports, driver's licenses, and biometric data in the form of verifiable credentials. The Agent allows the subject to establish unique, confidential, private and authentic channels with other agents.
- Verifiable Data Registry (VDR) - A storage location where information relating to Decentralized Identifiers (DIDs) and credential metadata are stored. Permissionless blockchains or permissioned distributed ledger networks can be used to facilitate the  discovery and resolution of DIDs and credential information.
- Trust Registry: Trust registries are services that help determine the authenticity and authorization of entities in an ecosystem governance framework. They allow governing authorities to specify what actions are authorized for governed parties and enable checking if an issuer is authorized to issue a particular credential type. Essentially, trust registries serve as a trusted source for verifying the legitimacy of credential issuers, wallet apps, and verifiers.

# Mapping a DID to the DNS

The W3C DID Core spec supports multiple ways of associating a DID to a domain.

alsoKnownAs: The assertion that two or more DIDs (or other types of URI, such as a domain name) refer to the same DID subject can be made using the [alsoKnownAs](https://www.w3.org/TR/did-core/#also-known-as) property.

Services: Alternatively, [services](https://www.w3.org/TR/did-core/#services) are used in DID documents to express ways of communicating with the DID subject or associated entities. In this case we are referring specifically to the "LinkedDomains" service type.

However, this association stemming only from the DID is unidirectional. By leveraging URI records as outlined in [DID in the DNS](https://datatracker.ietf.org/doc/html/draft-mayrhofer-did-dns-05#section-2), we can create a bidirectional relationship, allowing a domain to publish their associated DIDs in the DNS.

***Ex: _did.example-issuer.ca IN URI 1 0 “did:sov:XXXXXXX”***

This relationship enhances security, as an entity would require control over both the DID and the domain’s DNS server to create this bidirectional association, reducing the likelihood of malicious impersonation.

The ability for an organization to publish a list of their DIDs on the DNS is also beneficial as it establishes a link between the DNS, which is ubiquitously supported, and the distributed ledger (or other places where) the DIDs resides on which may not have the same degree of access or support, enhancing discoverability.

## URI record scoping

- The records MUST be scoped by setting the global underscore name of the URI RRset to ‘_did’ (0x5F 0x64 0x69 0x64).

## Issuer Handles

An issuer may have multiple sub entities issuing credentials on their behalf, such as the different faculties in a university issuing diplomas. Each of these entities will need to be registered separately in a trust registry and will likely have one or more DIDs of their own. For this reason, the introduction of an issuer handle, represented as a subdomain in the resource record name, provides a simple way to facilitate the distinction of DIDs, their public keys, and credentials they issue in their relationship to the issuer.

***Ex: _did.diplomas.university-issuer.ca IN URI 1 0 “did:sov:XXXXXXX”***

***Ex: _did.certificates.university-issuer.ca IN URI 1 0 “did:sov:XXXXXXX”***

# DID Public Keys in the DNS

The DID to DNS mapping illustrated in section 4 provides a way of showing the association between a DID and a domain, but no way of verifying that relationship. By hosting the public keys of that DID in its related domain’s zone, we can provide a cryptographic linkage to bolster this relationship while also providing access to the DID’s public keys outside of the distributed ledger where it resides, facilitating interoperability.

[TLSA records](https://www.rfc-editor.org/rfc/rfc6698) provide a simple way of hosting cryptographic information in the DNS.

## TLSA Record Scoping, Selector Field

When public keys related to DIDs are published in the DNS as TLSA records:

- The records MUST be scoped by setting the global underscore name of the TLSA RRset to ‘_did’ (0x5F 0x64 0x69 0x64).
- The Selector Field of the TLSA record must be set to 1, SubjectPublicKeyInfo: DER-encoded binary structure as defined in [RFC5280](https://www.rfc-editor.org/rfc/rfc5280).

## Issuer Handles

As mentioned in section 4.2, an issuer may have multiple sub entities issuing credentials on their behalf, likely with their own set/s of keypairs. Because these keypairs will need to be registered in a trust registry, and represented in the DNS as TLSA records, the use of an issuer Handle as outlined in section 4.2 will facilitate the distinction of the different public keys in their relation to the issuer.

***Ex: _did.diplomas.university-issuer.ca IN TLSA 3 0 0 “4e18ac22c00fb9...b96270a7b2”***

***Ex: _did.certificates. university -issuer.ca IN TLSA 3 0 0 “4e18ac22c00fb9...b96270a7b3”***

## Instances of Multiple Key Pairs

Depending on the needs of the issuer, it is possible they may use multiple keypairs associated with a single DID to sign and issue credentials. In this case a mechanism to differentiate which verificationMethod the public key is related to will need to be added to the name of the TLSA RRset.

A simple solution would be to create a standardized naming convention by expanding the RRset name using the fragment of the target verificationMethod's ID.

***Ex: _did.key-1.example-issuer.ca IN TLSA 3 0 0 "4e18ac22c00fb9...b96270a7b2"***

***Ex: _did.key-2.example-issuer.ca in TLSA 3 0 0 “4e18ac22c00fb9...b96270a7b3”***

## Benefits of Public Keys in the DNS

Hosting the public keys in TLSA records provides a stronger mechanism for the verifier to verify the issuer with, as they are able to perform a cryptographic challenge against the DID using the corresponding TLSA records, or against the domain using the corresponding [verificationMethod](https://www.w3.org/TR/did-core/#verification-methods) in the DID document. The accessibility of the public keys is also beneficial, as the verifier only needs to resolve the DID document on the distributed ledger and can perform the remainder of the cryptographic verification process using data available in the DNS, potentially limiting the burden of having to interoperate with a multitude of different distributed ledger technologies and transactions for key access.

# Digital Credential Verification using DIDs and the DNS

By leveraging the records and relationships outlined above, the verifier can verify a digital credential claim by:

- Looking up the credential’s issuer’s DID on a distributed ledger to resolve a DID document.
- Extracting the issuer’s domain from that DID document.
- Querying that domain for a _did URI record to confirm the issuer’s domain is also claiming ownership over that DID.
- Performing a verification of the credential’s signature/proof using the relevant verificationMethod in the DID document.
- Querying that domain for a _did TLSA record to confirm the public key expressed by the verificationMethod is also expressed by the issuer’s domain.

Through this process, the Verifier has not only cryptographically verified the credential they are being presented with, but also associated the issuing DID to a publicly resolvable domain, confirming it’s validity both semantically and cryptographically.

# Role of DNSSEC for Assurance and Revocation

It is a MUST that all the participants in this digital identity ecosystem enable DNSSEC signing for all the DNS instances they operate. See [RFC 9364 - DNS Security Extensions (DNSSEC)](https://datatracker.ietf.org/doc/html/rfc9364).

DNSSEC provides cryptographic assurance that the DNS records returned in response to a query are authentic and have not been tampered with. This assurance within the context of the _did URI and _did TLSA records provides another mechanism to ensure the integrity of the DID and its public keys outside of the distributed ledger it resides on directly from the domain of its owner.

Within this use-case, DNSSEC also provides revocation checks for both DIDs and public keys. In particular, a DNS query for a specific _did URI record or _did TLSA record can return an [NXDOMAIN](https://www.rfc-editor.org/rfc/rfc8020) response if the DID or public key has been revoked. This approach can simplify the process of verifying the validity of DIDs and public keys by reducing the need for complex revocation mechanisms or implementation specific technologies.

# The Role of Trust Registries in Bidirectional Credential Verification

A trust registry is a decentralized system that enables the verification of the authenticity and trustworthiness of issuers of digital credentials. Trust registries can be implemented using distributed ledger technology and leverage the DNS, to provide a transparent and auditable record of issuer information.

When an entity is presented with a verifiable claim, there are three things they will want to ensure:

1. That a claim hasn’t been altered/falsified at any point in time, via cryptographic verifiability and Verifiable Data Registries (VDRs).
2. That a claim has accurate representation via authentication, via DID Discovery & mapping within DNS as described above.
3. That a claim has authority, or in other words, does the issuer have authority in its issuance of credentials, via the use of trust registries (trust lists)

In this memo, Trust registries enable the verification of the authority of an issuer and by extent their credentials. The role of a trust registry within the context of this document is to confirm the authenticity and trustworthiness of the issuer to the verifier after they have validated the digital credential using the mechanisms described previously. This involves the trust registry taking on the role of a trust anchor for a given ecosystem, providing input for the verifier’s ultimate trust decision regarding the credential they are being presented with. The assumption is made that the trust registry would be operated under the authority of an institution or organization such that their claims and input to the trust decision would be considered significant or definitive. An example of such an organization would be a government entity in relation to the issuance of a Driver’s Licence.

It is important to note that the DNS based trust registry mechanism described in this section is not meant to operate in place of an alternative implementation but provide an easy to implement and use mechanism to extend such a solution.

This section also does not describe the process of the trust registry’s verification of an issuer, or the process of how an issuer would become accredited by or join a trust registry.

## Issuer's Membership Claim in a Trust Registry

Once the verifier has successfully completed the credential verification process outlined in section 6, they have definitive proof that the credential they are being presented with was issued by the claimed issuer, and that issuer can be resolved to an organization’s or entity’s domain. However, this process does not provide definitive proof the issuer is to be trusted or has the authority to issue such a credential. The issuer, through use of URI records and the _trustregistry label, can assert the claim that they are a member of a trust registry.

***Ex: _trustregistry.example-issuer.ca IN URI 0 1 “example-trustregistry.ca”***

This record indicates the verifier can then query the “example-trustregistry.ca” for further URI and TLSA records proving “example-issuer.ca”s membership.

### URI Record Name Scoping

When trust registry membership claims are published in the DNS

- The records MUST be scoped by setting the global (highest-level) underscore name of the URI RRset to ‘_trustregistry’ (0x5F 0x74 0x72 0x75 0x73 0x74 0x72 0x65 0x67 0x69 0x73 0x74 0x72 0x79)

## Trust Registry Membership Proof

The Trust Registry can assert an issuer's membership using TLSA records in a similar fashion to the methods outlined by section 5.1.

***Ex: _example-issuer.ca._trustregistration.example-trustregistry.ca in TLSA 3 0 0 “4e18ac22c00fb9...b96270a7b2”***

Note that the first component of the URI is the issuer’s domain, followed by the _trustregistration label. This combination indicates that the domain expressed is registered by this trust registry as per its governance model, and this is their public key. This association created by the TLSA record effectively has created a chain of trust, beginning at the DID’s verificationMethod, continuing to the issuer’s domain, and finally resolving at the Trust Registry.

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

# References

DIACC – TR Document

- Trust Over IP (ToIP) working group
- [ToIP Trust Registry Specification V1](https://github.com/trustoverip/tswg-trust-registry-tf/blob/main/v1/docs/ToIP%20Trust%20Registry%20V1%20Specification.md)

Pan-Canadian Trust Framework

- PCTF Trust Registries Draft Recommendation V1.0 DIACC / PCTF13
- [PCTF Trust Registries Component Overview Discussion Draft V0.02](https://diacc.ca/wp-content/uploads/2023/03/PCTF-Trust-Registries-Component-Overview_Draft-Recomendation-V1.0.pdf)

Decentralized Identity Foundation (DIF) Credentials Working Group

- [https://trustoverip.github.io/essiflab/glossary](https://essif-lab.eu )

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
