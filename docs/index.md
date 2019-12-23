---
title: "edi3 Notary Service 1.0 Specification"
specID: "notary/1"
status: "![raw](http://rfc.unprotocols.org/spec:2/COSS/raw.svg)"
editors: "[Shujing Liew](mailto:LIEW_Shujing@imda.gov.sg)"
contributors: Raymond, Rui Jie
---

# Abstract

This document describes the technical specifications for notarisation of trade documents using open-source solution leveraging on Distributed Ledger Technology (DLT). This allows end users receiving the document to verify the provenance and integrity of the document. 

The use of such technology offers several advantages. 

- There is no proprietary lock-in; solution will be open-source which lowers barriers of entry to market

- There is no centralised infrastructure (governance/sovereignty); thus, there would not be any single entity in charge of the verification/data that eliminates the trust concern of which authority will have all the trade information in trade world especially when cross-border trading is involved

- Elevates automated trade document processing; allows for any parties in the supply chain to issue trade documents, and for anyone to quickly check the validity of a digital trade documents. This will in time make trade facilitation easier and smoother for the whole supply chain workflow

- Connecting the disconnected world (digital standard); allowing isolated eco-systems to be interoperable
_put a short summary of the spec here_

# Introduction

Before digital transformation transpires, trade documents require a lot of administrative burden resulting in an increase of time and cost of manual verification. With the emerging digital technologies in the trade world, it has changed the way we work and helped to improve operational efficiency across supply chains.

Trade has grown remarkably over the last century with the trade world having numerous isolated ecosystems. Such system isolations resulted in individual systems not being able to verify trade documents from other ecosystems.

Lack of trust in the data is another major contributing factor that hinders global trade facilitation and the move to true digitalization of the supply chain. Paperless initiatives are yet to be widely adopted. For example, in sea freight, Bill of Lading is still made up of three originals and require six copies and digitized formats are still unacceptable for many parties.[1]

There has always been an inherent lack of trust between buyers, sellers, supply chain participants, agencies and governments. Due to this layer of trust, work processes such as document verification is still heavily reliant on the use of papers.[2]

Current forms of digital document are neither tamper-proof nor have provenance, resulting in digitalised documents still falling back to the paper documents at some point of the supply chain flow. Thus, we need to have a solution that allows trade documents to be cryptographically trustworthy and can be verified independently.

A major cultural and paradigm shift in the trade world has taken shape since the introduction of Blockchain almost a decade ago. Blockchain, which is a form of DLT offers opportunities to increase reliability and security of trade transactions.[3]

As we embark into the Blockchain to facilitate greater transparency, portability of information and greater operational efficiency, trade documents can now be notarised digitally and allowed tracing on the provenance and verification of digitally issued documents. This notarisation makes use of a framework to format data so that it can be fingerprinted, and then notarised on a trusted platform, such as the Ethereum blockchain.


## Goals

- Standard wrapper for document (see JSON schema 3)
- Support multiple document types
- Verify issuer’s identity (provenance)
- Tamper-proof document integrity (integrity)
- Selective data disclosure (data obfuscation)
- Standard & best-practices for storage & transmission 
- Support multiple backend (ETH/API/Private Blockchain)
- Compatible with W3C Verifiable Claims


## Use Cases

#### Non-transferable Document

##### Certificate of Non-Manipulation

<diagram>

###### Preconditions
- The issuer has a domain name which is custom.example.com.
- The issuer has deployed a document store smart contract on Ethereum.

###### Prove Ownership of Document Store
- Create TXT record to bind document store to domain.

###### Prove Document Issuance
- Publish the merkle root of the document onto the document store.

###### Verification
- Issuance Method
  - Check that merkle root of the document exists on the ‘Issued’ list on the document store.
  - Check that the merkle root of the document does not exist on the revocation (‘Revoked’) list of the document store.

- Integrity Method
  - Check that the document data’s hash is the ‘targetHash’;
  - Check the targetHash resolves to the merkle root using the proof (if any);

- Issuer Identity
  - Check that a TXT record exist on the domain claiming the ownership of the document store

#### Transferable Document
##### Bill of Lading

<diagram>

###### Preconditions
- The issuer has a domain name which is custom.example.com.
- The issuer has deployed a token registry smart contract on Ethereum.

###### Prove Ownership of Token Registry
- Create TXT record to bind token registry to domain.

###### Prove Document Issuance
- Assign an owner to the corresponding document’s target hash.

###### Verification

- Issuance Method
  - Check that target hash of the document exists in the list of tokens in the token registry.

- Integrity Method
  - Check that the document data’s hash is the ‘targetHash’.
  - Check the targetHash resolves to the merkle root using the proof (if any).
  
- Issuer Identity
  - Check that a TXT record exists on the domain claiming the ownership of the token registry.


# Data Model

## JSON Schema

The document data model is defined using JSON Schema with the following definitions found in Annex A.


## JSON Schema Validator

A useful online tool JSON Schema Validator that can help to validate whether the document data conforms to the schema: https://www.jsonschemavalidator.net/

Just paste the content of the schema document on the left panel and enter the document data on the right. The tool will instantly validate the document String/Object and notify any potential errors.

An example of a passing validation:

<diagram>

An example of a failed validation:

<diagram>

# Basic Concept
## Verification Method

The purpose of the verifier is to provide a generic verification method to verify OpenAttestation(OA) documents. The verifier will provide default verification methods conforming to the standard verification process proposed in OpenAttestation yet providing opportunities for it to be extended.

## Overview of the verification methods

<diagram>

A verifier is made up of multiple Verification Methods. In the diagram above, OpenAttestationDnsTxt, OpenAttestationEthereumDocumentStoreIssued and OpenAttestationHash are examples of Verification Methods provided.

The role of a verification method is to verify the OA document is valid against specific criterias. Since there are many types and versions of OA document, not all test should run against all types of OA document. For that reason, a test method is also defined to test if a method should run against a document. If the method is incompatible with the document type, it should skip the method.

# Abstract Semantics

## Notary Process

## Notary Information Model

# Technology Bindings

## Ethereum


## REST API

 
# References

_various reference stuff that readers might need to know_

## Intellectual Property

All material published on edi3.org including all parts of this specification remain the intellectual property of the contributor.  However, contributors should participate on the basis that the specification IP may be incoprporated into a UN/CEFACT standard and, in that case, the IP rights will transfer to the UN as per the [UN/CEFACT IPR Policy](https://www.unece.org/fileadmin/DAM/cefact/cf_plenary/plenary12/ECE_TRADE_C_CEFACT_2010_20_Rev2E_UpdatedIPRpolicy.pdf).

## License

This Specification is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 3 of the License, or (at your option) any later version. See http://www.gnu.org/licenses.
 
## Change Process

 This document is governed by the [2/COSS](http://rfc.unprotocols.org/spec:2/COSS/) (COSS).

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" 
in this document are to be interpreted as described in RFC 2119.

## Glossary

Phrase | Definition
------------ | -------------
|

## Dependencies

This specification depends on the following specifications

* spec 1
* spec 2

The following specifications depends on this specification
 
* spec 1
* spec 2

## Related Information

_put a list of any relevant referneces here, and link to them from the appropriate part of the specification_
