---
title: "Workload Identifier"
abbrev: "Workload Identifier"
category: std

docname: draft-rosomakho-wimse-identifier-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Workload Identity in Multi System Environments"
keyword:
 - workload
 - identifier
venue:
  group: "Workload Identity in Multi System Environments"
  type: ""
  mail: "wimse@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/wimse/"
  github: "yaroslavros/wimse-identifier"
  latest: "https://yaroslavros.github.io/wimse-identifier/draft-rosomakho-wimse-identifier.html"

author:
 -
    ins: Y. Rosomakho
    name: Yaroslav Rosomakho
    organization: Zscaler
    email: yaroslavros@gmail.com
 -
    ins: J. Salowey
    name: Joe Salowey
    organization: CyberArk
    email: joe@salowey.net

normative:

informative:


--- abstract

This document defines a canonical identifier for workloads, referred to as the Workload Identifier. A Workload Identifier is a URI that uniquely identifies a workload within the context of a specific trust domain. This identifier can be embedded in digital credentials, including X.509 certificates and security tokens, to support authentication, authorization, and policy enforcement across diverse systems. The Workload Identifier format ensures interoperability, facilitates secure identity federation, and enables consistent identity semantics.


--- middle

# Introduction

In modern distributed systems, workloads such as services, applications, or containerised tasks require cryptographically verifiable identities to support secure communication, access control, and auditability. As systems scale across trust domains, administrative boundaries, and heterogeneous platforms, the need for a consistent and interoperable identifier format becomes critical.

This document defines the Workload Identifier, a URI-based {{!URI=RFC3986}} identifier intended to uniquely represent a workload within the context of an issuing authority. The identifier is designed to be stable, globally unique within a given trust domain, and suitable for use in digital credentials such as X.509 certificates , JSON Web Tokens (JWTs, {{?JWT=RFC7519}}), and other security artifacts.

The Workload Identifier format is simple yet expressive. It enables organisations to define trust boundaries, delegate identity management, and reason about workloads in a uniform way across service meshes, cloud environments, and on-premises infrastructure. This specification is intended to be generic and reusable beyond the context of any single system or architecture, including but not limited to the Workload Identity in Multi-System Environments (WIMSE) architecture {{?ARCH=I-D.ietf-wimse-arch}}.

The primary goals of this specification are:

- To define the syntax and semantics of a Workload Identifier.
- To establish requirements for issuers and consumers of such identifiers.
- To promote interoperability across different identity systems and domains.

This document does not prescribe how identifiers are issued or verified. Instead, it focuses on the identifier’s format, uniqueness guarantees, and its relationship to trust domains.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Terminology

The following terms are used throughout this document:

Workload:

: An independently addressable and executable software entity. This may include microservices, containers, virtual machines, serverless functions, or similar components that initiate or receive network communications.

Workload Identifier:

: A URI-based identifier that uniquely represents a workload within a specific trust domain. It is intended to be included in security credentials and interpreted within the scope of an issuing authority.

Trust Domain:

: A security boundary defined and controlled by a single administrative authority. A trust domain establishes its own rules for identity issuance, validation, and policy enforcement.

Issuer:

: An entity responsible for assigning and validating Workload Identifiers.

Consumer:

: An entity that evaluates or verifies a Workload Identifier, typically as part of authentication or authorisation decisions. This includes relying parties, verifiers, and policy enforcement points.

# Workload Identifier Specification

A Workload Identifier is a URI {{URI}} that uniquely identifies a workload. It encodes both the trust domain and a workload-specific path, enabling unambiguous identification of workloads across administrative and organisational boundaries.

The identifier is designed to be stable and suitable for inclusion in digital credentials such as X.509 certificates and security tokens. This section defines the format, structure, and associated requirements for Workload Identifiers.

## URI Requirements

A Workload Identifier MUST be an absolute URI, as defined in {{Section 4.3 of URI}}. The URI format allows different schemes (e.g., `spiffe`, `https`, or `urn`) depending on deployment requirements. Example identifiers:

~~~
spiffe://incubation.example.org/ns/experimental/analytics/ingest
https://trust.corp.example.com/workload/af3e86cb-7013-4e33-b717-11c4edd25679
urn:example:container:db-read-replica
~~~

## Scheme Specific Portion

The format and semantics scheme specific part of the URI that follows the identity is determined by the issuer in the trust domain. What the identity refers to is also determined by the issuer. For example a workload identity may refer to a specific instance of a running piece of software or it may refer just to a specific software version running in a particular environment, or it may refer to the role that the software performs within the system.  The scheme specific part of the URI may just be an opaque unique identifier used to look up the additional identity information in another system. Some examples of these concepts are given below:

* Opaque identifier

~~~
spiffe://prod.trust.domain/89a6ec51-f877-44c0-9501-b213597f2d1d
~~~

* Application role

~~~
spiffe://prod.trust.domain/ns/prod-01/sa/foo-service
~~~

* Specific instance of application role

~~~
spiffe://prod.trust.domain/ns/prod-01/sa/foo-service/iid-
      1f814646-87b5-4e26-bb55-1d13caccdd8d
~~~

* Specific code for an application role

~~~
spiffe://prod.trust.domaain/foo-servce#@sha256:
      c4dbb1a06030e142cb0ed4be61421967618289a19c0c7760bdd745ac67779ca7
~~~

Other concepts may be defined within the trust domain depending on what is important in the system and what information is available when the identity is issued. A trust domain should define the scheme specific portion of the URI to meet their auditing and authorization needs.

## Trust Domain Association

The authority component of the URI defines the trust domain which is responsible for issuing, validating, and managing Workload Identifiers within its scope.

Workload Identifiers are interpreted in the context of the trust domain that issued the credential. Identifiers with identical path components but different trust domains represent different workloads.

Issuers within a trust domain MUST ensure uniqueness of all Workload Identifiers they assign.

## Stability and Uniqueness

Workload Identifiers are intended to be stable over time. An identifier assigned to a specific workload should not be reassigned to a different workload unless explicitly intended by the policies of the trust domain.

Workload Identifiers are globally unique when the trust domain is globally unique. This is typically achieved by using a fully qualified domain name (FQDN) under organisational control.

For example, the following contains identifiers of two distinct globally unique Workload Identifiers

~~~
spiffe://dev.example.com/ns/default/database/backend
spiffe://prod.example.com/ns/default/database/backend
~~~

# Usage in Credentials and Tokens

Workload Identifiers are designed to be embedded in cryptographic credentials and security tokens that are used to assert the identity of workloads during authentication, authorisation, and auditing. This section describes how such identifiers may be represented in commonly used formats.

## X.509 Certificates

Workload Identifier included in an X.509 are encoded in the subject alternative name extension as a URI using the uniformResourceIdentifier field, as defined in {{Section 4.2.1.6 of ?X509-PROFILE=RFC5280}}.

For example,

~~~
X509v3 extensions:
    X509v3 Subject Alternative Name:
        URI:spiffe://example.org/ns/default/analytics/ingest
~~~

Consumers MUST NOT attempt to interpret or derive workload identity from other certificate fields such as the Common Name.

## JSON Web Tokens (JWT)

When a Workload Identifier is included in a JWT, it MUST appear in the "sub" (Subject) claim, as defined in {{Section 4.1.2 of JWT}}.

## Interpretation by Consumers

Consumers of credentials and tokens MUST validate that the Workload Identifier is consistent with the expected trust domain and issuing authority. Consumers SHOULD NOT make assumptions about internal structure or semantics of the identifier beyond the URI format defined in this specification.

For authorisation decisions, consumers may map Workload Identifiers to policies or roles. However, such mappings are out of scope for this specification.

# Security Considerations

The Workload Identifier is intended to be used as a stable, verifiable identity for workloads. Its use in cryptographic credentials means it must be protected against spoofing, ambiguity, and misinterpretation. This section outlines security considerations for issuers, consumers, and system designers.

## Identifier Authenticity

Workload Identifiers MUST only be considered valid when presented in a credential or token that has been cryptographically verified. An identifier received outside such a context, such as a plaintext string in a request, MUST NOT be treated as authenticated.

Consumers MUST verify the signature, issuer, and validity of the credential or token before considering Workload Identifier as authenticated.

## Trust Domain Validation

Consumers MUST validate that the trust domain in the Workload Identifier matches an expected or explicitly trusted domain. Failure to do so may allow identifiers from unauthorised domains to be accepted as legitimate.

Where appropriate, consumers should maintain an allowlist of trusted domains or trusted issuing authorities.

## Identifier Reuse and Collision

Issuers SHOULD ensure that Workload Identifiers are not reused across different workloads unless such reuse is intentional and well-scoped. Reassignment of identifiers to unrelated entities can result in privilege escalation or confusion in audit trails.

Consumers SHOULD assume that identifiers are permanent within their domain of interpretation and treat unexpected reuse with suspicion.

## Information Disclosure

Because Workload Identifiers may encode topological or semantic information, they may inadvertently reveal deployment details. Issuers and system designers should take care not to expose sensitive naming conventions in externally visible identifiers.

Where possible, identifier paths SHOULD be minimally descriptive and avoid exposing internal implementation details unless necessary for interoperation.

## Wildcard and Prefix Matching

Consumers SHOULD NOT interpret Workload Identifiers using wildcard or prefix matching unless explicitly specified by policy. For example, treating all identifiers under prefix of `spiffe://example.org/ns/db/` as equivalent may lead to incorrect authorisation.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
