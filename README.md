# Practical Implementation Guide for Regulation (EU) 2020/1056 (eFTI)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Regulation](https://img.shields.io/badge/eFTI-EU%202020%2F1056-blue.svg)](https://eur-lex.europa.eu/eli/reg/2020/1056/oj)
[![Delegated Act](https://img.shields.io/badge/Delegated%20Act-EU%202024%2F2529-green.svg)](https://eur-lex.europa.eu/eli/reg_del/2024/2529/oj)
[![JCS Standard](https://img.shields.io/badge/RFC-8785-orange.svg)](https://datatracker.ietf.org/doc/html/rfc8785)
[![Portal](https://img.shields.io/badge/Portal-CLTD%20European%20Digital%20Trade%20Center-blue)](https://solomonbrokerage.com/cltd/)

## Mandatory Legal Attribution Notice (Estonian Copyright Act AutÕS § 12)

> **Mandatory Attribution Notice:**  
> Engineered under the DTLMF Architecture Blueprint. Original Concept and Intellectual Property © 2025–2026 Solomon Brokerage OÜ (Tallinn, Republic of Estonia). Sole Founder & Strategic Architect: Alejandro De La Parra Solomon. Licensed under MIT / CC BY 4.0.

## Regulatory Non-Affiliation Disclaimer

> **DISCLAIMER:**  
> DTLMF operates strictly as a voluntary B2B trade finance enablement utility outside the mandatory B2A platform certification scope of Regulation (EU) 2020/1056 Articles 11–13.

---

## Executive Overview

This repository provides an authoritative technical implementation roadmap and reference architecture for European freight forwarders, logistics software vendors, and transport authorities preparing for **Regulation (EU) 2020/1056 (electronic Freight Transport Information - eFTI)** and its associated **Delegated Act (EU) 2024/2529**.

---

## Production Core Repositories & Live Portal

This implementation framework references our production open-source engines:

* **Layer 1 Substrate Compliance Core:** [`polkadot-efti-compliance-core`](https://github.com/adelaparras/polkadot-efti-compliance-core) – Substrate FRAME pallets (`pallet-efti-anchor-vc`, `pallet-did-registry`) for immutable document anchoring.
* **Layer 2 Arbitrum Settlement Engine:** [`arbitrum-shariah-settlement-engine`](https://github.com/adelaparras/arbitrum-shariah-settlement-engine) – Shariah-compliant AAOIFI EVM smart contracts.
* **Client-Side Canonicalizer SDK:** [`dtlmf-efti-jcs-sdk`](https://github.com/adelaparras/dtlmf-efti-jcs-sdk) – Open TypeScript SDK for RFC 8785 JSON canonicalization and SHA-256 digest generation.
* **European Digital Trade Center (CLTD) Portal:** [https://solomonbrokerage.com/cltd/](https://solomonbrokerage.com/cltd/)

---

## The 4-Phase eFTI Roadmap to Mandatory Certification (July 2027)

```
+-------------------------------------------------------------------------------------------------+
|                                 eFTI Regulatory Implementation Timeline                         |
+-------------------------------------------------------------------------------------------------+
|                                                                                                 |
|   Phase 1: Baseline Architecture & Sandbox Testing                      [2024 - Q2 2025]        |
|   - Adoption of Delegated Act (EU) 2024/2529 common data subsets                                 |
|   - Prototyping Access Point Gateways (APGs) and RFC 8785 canonical hashing                      |
|                                                                                                 |
|   Phase 2: Voluntary Corridor Pilots (EE, FI, DE Fastlanes)             [Q3 2025 - Q1 2026]     |
|   - Live mTLS 1.3 Authority Access Gateway (AAG) query flow integration                          |
|   - Verification of bilateral transport manifests with national customs authorities             |
|                                                                                                 |
|   Phase 3: Formal CAB Accreditation & Audit Readiness                   [Q2 2026 - Q4 2026]     |
|   - Conformity Assessment Body (CAB) audits under ISO/IEC 17065 & eFTI technical specifications |
|   - Hardware Security Module (HSM/KMS) root key attestation                                     |
|                                                                                                 |
|   Phase 4: Full Mandatory Enforcement & Production Mainnet              [July 2027 ONWARD]      |
|   - Article 5 obligation: EU competent authorities MUST accept eFTI data digitally               |
|   - Mandatory certification of commercial eFTI platforms under Articles 11-13                   |
+-------------------------------------------------------------------------------------------------+
```

---

## Delegated Act (EU) 2024/2529 Data Sets & JSON-LD Schemas

Delegated Act (EU) 2024/2529 establishes the standardized eFTI data dictionary, bifurcated into:

1. **Dataset A (Transport Operations & Modalities):**
   - Road (CMR), Rail (CIM), Inland Waterway (CMNI), Air (Montreal Convention), and Maritime (FAL/e-BL).
   - Core fields: Consignment identification, carrier identity, gross vehicle mass, route plan, dangerous goods classifications (ADR/RID).

2. **Dataset B (Commercial & Supply Chain Supplementary Data):**
   - Commercial invoice identifiers, Incoterms 2020 delivery terms, commodity codes (Harmonized System - HS Codes), cargo insurance coverage, and eIDAS qualified electronic timestamps.

---

## JCS RFC 8785 Canonicalization & SHA-256 Digest Pipeline

Under eFTI specifications, disparate software runtimes must generate bit-for-bit identical hashes from identical business data. This requires **JSON Canonicalization Scheme (JCS - RFC 8785)**:

```
[Raw Freight Manifest JSON]
            |
            v
[RFC 8785 Normalization Engine]
   * Lexicographical key sorting (UTF-16 code units)
   * Deterministic IEEE 754 number representation
   * Whitespace elimination outside string literals
            |
            v
[Deterministic Canonical JSON String]
            |
            v
[SHA-256 Cryptographic Hash] ===> 32-Byte Root Digest (`0x4f82a...`)
            |
            v
[Anchored into pallet-efti-anchor-vc on Substrate Layer 1]
```

Utilize the [`dtlmf-efti-jcs-sdk`](https://github.com/adelaparras/dtlmf-efti-jcs-sdk) package to execute this pipeline in Node.js, TypeScript, or browser runtimes.

---

## Authority Access Gateway (AAG) 3-Step mTLS 1.3 Inspection Flow

For customs officers, road police, and maritime port inspectors in Member State fastlanes (Estonia, Finland, Germany):

```
+--------------------------------------------------------------------------------------------------+
|                   Step 1: Mutual TLS 1.3 Handshake & Authority Identity Validation               |
|   Customs Inspector Client <================ (mTLS 1.3 / eIDAS QES) =================> AAG Gateway|
|   - Client verifies AAG server certificate issued by trusted EU Qualified Trust Service Provider  |
|   - AAG verifies Inspector client certificate against EU trusted list of competent authorities   |
+--------------------------------------------------------------------------------------------------+
                                                |
                                                v
+--------------------------------------------------------------------------------------------------+
|                   Step 2: Scoped Extrinsic Query & Decryption Authorization                      |
|   - Inspector presents consignment movement ID: `EFTI-EU-2026-0098412`                           |
|   - AAG queries Substrate Layer 1 (`pallet-efti-anchor-vc`) for unrevoked on-chain digest        |
|   - Verified DID registry confirms carrier credential validity via KILT XCMP dispatcher          |
+--------------------------------------------------------------------------------------------------+
                                                |
                                                v
+--------------------------------------------------------------------------------------------------+
|                   Step 3: Just-in-Time Data Delivery & Zero-Trace Minimization                   |
|   - AAG serves the canonical JSON-LD manifest payload over encrypted session                     |
|   - Inspector confirms root digest equality: SHA256(JCS(Manifest)) === Anchored_Digest           |
|   - Zero persistent PII retained on gateway; complete GDPR Article 5(1)(c) data minimization     |
+--------------------------------------------------------------------------------------------------+
```
