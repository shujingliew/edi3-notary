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


# Introduction

Before digital transformation transpires, trade documents require a lot of administrative burden resulting in an increase of time and cost of manual verification. With the emerging digital technologies in the trade world, it has changed the way we work and helped to improve operational efficiency across supply chains.

Trade has grown remarkably over the last century with the trade world having numerous isolated ecosystems. Such system isolations resulted in individual systems not being able to verify trade documents from other ecosystems.

Lack of trust in the data is another major contributing factor that hinders global trade facilitation and the move to true digitalization of the supply chain. Paperless initiatives are yet to be widely adopted. For example, in sea freight, Bill of Lading is still made up of three originals and require six copies and digitized formats are still unacceptable for many parties.<sup id="footnote1">[[1]](#f1)</sup> 

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

![docs](document_store.png)

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

![docs](token_registry.png)

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

![docs](json_success.PNG)

An example of a failed validation:


![docs](json_failed.PNG)


# Basic Concept
## Verification Method

The purpose of the verifier is to provide a generic verification method to verify OpenAttestation(OA) documents. The verifier will provide default verification methods conforming to the standard verification process proposed in OpenAttestation yet providing opportunities for it to be extended.

### Overview of the verification methods

<diagram>

A verifier is made up of multiple `Verification Methods`. In the diagram above, `OpenAttestationDnsTxt, OpenAttestationEthereumDocumentStoreIssued` and `OpenAttestationHash` are examples of `Verification Methods` provided.

The role of a verification method is to verify the OA document is valid against specific criterias. Since there are many types and versions of OA document, not all test should run against all types of OA document. For that reason, a `test` method is also defined to test if a method should run against a document. If the method is incompatible with the document type, it should skip the method.

As a result, a verification method should implement 3 abstract methods: `verify, test and skip`.

The verification method should return a `VerificationFragment` which states the status of the verification. The status key can have the following values:

- `VALID`: when the verification is successful
- `INVALID`: when the verification is unsuccessful
- `ERROR`: when an unexpected error is met
- `SKIPPED`: when the verification was skipped by the manager

#### test(document: Document): boolean
This function takes in an OA document and returns a boolean result that determines if this verification method is meant to be ran against the document.

In the case that the test passes, we should run the `verify` function. Otherwise, the `skip` function should be ran.

#### skip(): VerificationFragment
This function should be ran when the verification method is skipped for the given OA document. It will return the skipped `VerificationFragment` for the test.

An example of `VerificationFragment` that skips the DNS-TXT verification:
{
  ```json
  "name": "OpenAttestationDnsTxt",
  "type": "ISSUER_IDENTITY",
  "status": "SKIPPED",
  "message": "Document issuers doesn't have \"documentStore\" or \"token\" property"
  
  ```
}

#### verify(document: Document): VerificationFragment

This function should be ran when the test passes. Running this function will execute the necessary computation to determine if the document passes the verification method.

An example `VerificationFragment` of a passing test:

```json
{
  "name": "OpenAttestationEthereumDocumentStoreIssued",
  "type": "DOCUMENT_STATUS",
  "status": "VALID",
  "data": {
    "details": [
      {
        "address": "0x007d40224f6562461633ccfbaffd359ebb2fc9ba",
        "issued": true
      }
    ],
    "issuedOnAll": true
  }
}
```

An example `VerificationFragment` of a failed test:

```json
{
  "name": "OpenAttestationEthereumDocumentStoreIssued",
  "type": "DOCUMENT_STATUS",
  "status": "INVALID",
  "message": "Certificate has not been issued",
  "data": {
    "details": [
      {
        "address": "0x20bc9C354A18C8178A713B9BcCFFaC2152b53990",
        "error": "call exception (address=\"0x20bc9C354A18C8178A713B9BcCFFaC2152b53990\", method=\"isIssued(bytes32)\", args=[\"0x85df2b4e905a82cf10c317df8f4b659b5cf38cc12bd5fbaffba5fc901ef0011b\"], version=4.0.40)",
        "issued": false
      }
    ],
    "issuedOnAll": false
  }
}
```

### Verification Types

A `Verification Type` is a type label to a verification method. It specifies what type of verification the method is performing. The diagram above shows the 3 default verification types: `ISSUER_IDENTITY`, `DOCUMENT_INTEGRITY` and `DOCUMENT_STATUS`.

For the validity of the verification type to be true, the requirements must be met:

1. At least one method in that type should return `VALID` as the status.
1. All methods in the type should return either `VALID` or `SKIPPED` as the status.

### Verifier

The verifier is a function used to verify any OpenAttestation document. It returns a set of `VerificationFragment` from the different verification methods.

From the `VerificationFragments` we can then determine if the OA document is valid. The OA document is valid when all `Verification Types` has valid statuses.

## Default Verification Types and Methods

### Document Integrity (DOCUMENT_INTEGRITY)

The `DOCUMENT_INTEGRITY` type of verification methods ensure that the content of the issued document has not been modified since the document has been issued, with exception of data which has been removed using built-in obfuscation mechanism.

#### OpenAttestationHash

The `OpenAttestationHash` method checks the integrity of the document using the `OpenAttestationSignature2018` method by digesting the content of the OA document and comparing it with the document's `targetHash`.

In addition, if the `targetHash` does not match the `merkleRoot` of the document, the function also checks that the `targetHash` resolves to the `merkleRoot` using the given `proofs`.

### Document Status (DOCUMENT_STATUS)

The `DOCUMENT_STATUS` type of verification methods checks that the document has been issued and that it's issuance status is in good standing. Methods of these types generally verifies the issuance status against a record maintained externally, ie Records on a Blockchain or API endpoints.

#### OpenAttestationEthereumDocumentStoreIssued

The `OpenAttestationEthereumDocumentStoreIssued` checks the issuance status of the document using `OpenAttestationSignature2018` with `DOCUMENT_STORE` or `TOKEN_REGISTRY` methods.

Signature of OA Document using this method:

```json
{
  "proof": {
    "type": "OpenAttestationSignature2018",
    "method": "DOCUMENT_STORE",
    "value": "0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d"
  }
}
```

Such document uses a smart contract to store the issuance status of the document.

In the case of `DOCUMENT_STORE` smart contract, the check calls the method `isIssued(merkleRoot)` implemented on the Ethereum smart contract. It returns a valid `VerificationFragment` if the merkle root of the document has been marked as issued.

In the case of `TOKEN_REGISTRY` smart contract, the check calls the method `ownerOf(targetHash)` implemented on the Ethereum smart contract. It returns a valid `VerificationFragment` if the owner of the document is non-zero.

#### OpenAttestationEthereumDocumentStoreRevoked

The `OpenAttestationEthereumDocumentStoreRevoked` checks the revocation status of the document using `OpenAttestationSignature2018` with `DOCUMENT_STORE` or `TOKEN_REGISTRY` methods, similar to `OpenAttestationEthereumDocumentStoreIssued`.

In the case of `DOCUMENT_STORE` smart contract, the check calls the method `isRevoked(merkleRoot)` implemented on the Ethereum smart contract. It returns a valid `VerificationFragment` if the merkle root of the document has not been marked as revoked.

### Issuer Identity (ISSUER_IDENTITY)

The `ISSUER_IDENTITY` type of verification methods checks and return the identity of the issuer. Methods of these types generally verify the identity of the issuer against a decentralised identity provider (ie DID, DNS, etc) or some centrally managed identity registry (ie Singapore's OpenCerts Registry, Citizen Identity Registry, Business Identity Registry, etc).

#### OpenAttestationDnsTxt

The `OpenAttestationDnsTxt` method checks the identity of a document issuer using the `DNS-TXT` identity proof mechanism. 

The sample provided below shows a document which claims that `openattestation.com` is the owner of the smart contract `0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d`, which in turn issued the said document:

```json
{
  "issuer": {
    "id": "https://openattestation.com",
    "name": "Open Attestation",
    "identityProof": {
      "type": "DNS-TXT",
      "location": "openattestation.com"
    }
  },
  "proof": {
    "type": "OpenAttestationSignature2018",
    "method": "DOCUMENT_STORE",
    "value": "0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d"
  }
}
```

The method checks against the DNS record of the domain to confirm the claimed relationship. A sample TXT record on `openattestation.com` to prove ownership of the contract `0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d` on Ethereum Ropsten network is provided below:

```
openatts net=ethereum netId=3 addr=0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d
```

Upon confirming that the matching DNS TXT record on the domain, the verification method returns a valid `VerificationFragment` as the result.

In depth discussion of DNS-TXT method is described [in this blog post](https://blog.gds-gov.tech/opencerts-2-0-decentralised-issuer-identity-verification-fb7e2cae8295)

## Usage

To use the default verifier:

```js
const document = "OA-Document";
const verificationFragments = verify(document);
const verified = isVerified(verificationFragments);
```

To extend the verifier with a custom name registry:

```js
const document = "OA-Document-With-Custom-Identity-Proof";
const customIdentityRegistry = "Custom-Verification-Method";
const verificationFragments = verificationBuilder(document, [
  ...defaultVerifiers,
  customIdentityRegistry
]);
const verified = isVerified(verificationFragments);
```

# Decentralised Document Rendering

## Status

Accepted

## Goal


![docs](decentralised_rendered_overview.png)

The purpose of the decentralised document renderer is to allow OpenAttestation (OA) document issuers to style their documents without code change to the different implementations of the document viewer. It does so by embedding the website specified by the document issuers as an iframe or webview (for mobile apps) and sending the content of the OA document into the iframe.

## Frame-to-Frame Communication

![docs](decentralised_rendered_api.png)

Since the decentralised renderer is rendering a document dynamically, `Actions` are used to two-way communication between the parent frame (document-renderer.com) and the child frame (renderer.example.com).

All actions follow the same structure and are composed of type and payload

- `type` indicates the kind of action executed, for instance, RENDER_DOCUMENT to render a document. The type of action is mandatory
- `payload` indicates optional data associated with the type, for instance, the content of the document to render.

Examples of actions used:

```js
const renderDocumentAction = {
  type: "RENDER_DOCUMENT",
  payload: {
    document: documentToRender
  }
};

const printAction = {
  type: "PRINT"
};
```

### From host to frame actions

The following list of actions are made for the host to communicate to the iframe (and thus must be handled by application embed in the iframe):

#### RENDER_DOCUMENT

This action sends the document data to the child frame to be rendered.

The payload is an object with 2 properties:

- document: (mandatory) document data as returned by getData method from @govtechsg/open-attestation
- rawDocument: (optional) Open Attestation document

Example:

```js
const action = {
  type: "RENDER_DOCUMENT",
  payload: {
    document: getData(document),
    rawDocument: document
  }
};
```

#### SELECT_TEMPLATE

This action selects a template amongst the one provided by the decentralized renderer (A renderer may provide 1 to many different templates to display a document).

The payload is the tab id to display.

Example:

```js
const action = {
  type: "SELECT_TEMPLATE",
  payload: "CUSTOM_TEMPLATE"
};
```

#### PRINT

This action request for the template to process the print action by the browser.

Example:

```js
const action = {
type: "PRINT"
const action = {
  type: "PRINT"
};
```

#### GET_TEMPLATES

This action returns the list of template tabs available on the document.

The payload is the document data as returned by `getData` method from @govtechsg/open-attestation

Example:

```js
const action = {
  type: "GET_TEMPLATES",
  payload: getData(document)
};
```

## From frame to host actions

The following list of actions are made for iframe to communicate to the host (and thus must be handled by application embedding the iframe):

### UPDATE_HEIGHT

This action provides the full content height of the iframe so that the host can adapt the automatically the size of the embedded iframe.

The payload is the full content height of the child iframe.

Example:

```js
const action = {
  type: "UPDATE_HEIGHT",
  payload: 150
};
```

### OBFUSCATE

This action provides the name of a field on the document to obfuscate. The value must follow path property.

The payload is the path to the field to be obfuscated.

Example:

```js
const action = {
  type: "OBFUSCATE",
  payload: "a[0].b.c"
};
```

### UPDATE_TEMPLATES

This action provides the list of templates that can be used to render a document. This is usually the response to the `GET_TEMPLATES` action.

Example:

```js
const action = {
  type: "UPDATE_TEMPLATES",
  payload: [
    {
      id: "certificate",
      label: "Certificate"
    },
    {
      id: "transcript",
      label: "Transcript"
    }
  ]
};
```

 
# References

OA Schema V3 - https://github.com/Open-Attestation/open-attestation/blob/56231b83b6bd0b6bb164bdd830fc02886b578ec1/src/schema/3.0/schema.json

Verifier Manager Model - https://github.com/Open-Attestation/oa-verify/pull/74

DNS Identity Proof - https://blog.gds-gov.tech/opencerts-2-0-decentralised-issuer-identity-verification-fb7e2cae8295

W3C VC - https://www.w3.org/TR/vc-data-model/

<sup id="f1">[1]</sup> [↩](#footnote1)
[UN/CEFACT White Paper on Data Pipeline](https://www.unece.org/fileadmin/DAM/cefact/GuidanceMaterials/WhitePaperDataPipeline_Eng.pdf) 

[2] Extracted from White Paper on Technical Application of Blockchain to UN/CEFACT deliverables by UN/CEFACT

[3] Extracted from ECE/TRADE/C/CEFACT/2019/8


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
