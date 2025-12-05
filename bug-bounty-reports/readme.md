## 🐛 Bug Bounty Reports

This folder contains a collection of vulnerability research and bug bounty findings submitted through HackerOne and private bounty programs.
All reports are fully redacted in compliance with program rules and do not disclose any sensitive information or target identities.

The purpose of this directory is to demonstrate:

 - Real-world security analysis

 - Real-world web application testing

 - Safe PoC construction

 - Professional report writing

 - Reconnaissance and methodology-driven testing

## 📌 Included Reports

### 1️⃣ Unauthenticated API DoS Vector

File: unauthenticated-api-dos-vector.md

Status: Submitted (pending re-evaluation)

Severity (Suggested): Medium → High

Summary:
Unauthenticated POST endpoints were found to process unvalidated payloads, leading to backend stalls (~135 seconds). This enables application-layer DoS without authentication.

### 2️⃣ GraphQL Schema Exposure via Error Responses

File: error-driven-graphql-schema-discovery.md

Status: Closed as Informational

Summary:
GraphQL introspection was disabled, but verbose error messages exposed hidden schema fields, mutation requirements, and authorization behavior. Useful for attack surface mapping.

## 🎯 Focus of These Reports

These reports highlight my skills in:

 - API Recon & Enumeration

 - Business Logic & Access Control Testing

 - GraphQL Error-based Enumeration

 - Authentication & Authorization Analysis

 - Resource Exhaustion / DoS Behavior Analysis

 - Safe Testing Methodology (no harmful payloads)

## 🔐 Ethical & Responsible Disclosure

All testing was performed:

 - Within program scope

 - On accounts owned by me

 - With safe, non-destructive payloads

 - Following HackerOne’s vulnerability disclosure guidelines

This repository intentionally redacts:

 - Company names

 - Hostnames

 - API paths

 - Sensitive identifiers

 - Any internal or user-related data

🧭 Purpose of This Folder

This section of my portfolio showcases:

 - My analytical thinking

 - My ability to chain and escalate findings

 - My practical experience with real-world systems

 - My report writing quality

 - My readiness for a Junior Pentester / Security Engineer role

If you'd like to jump into methodology, labs, or pentest case studies, check the root portfolio structure.
