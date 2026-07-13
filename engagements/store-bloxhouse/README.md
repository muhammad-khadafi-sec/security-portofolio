# 🛒 Store Bloxhouse

## Overview

Performed a white-box application security assessment against **store.bloxhouse.id**, focusing on invoice generation, payment workflows, API exposure, authorization controls, and business logic implementation.

The engagement aimed to identify vulnerabilities that could impact customer data confidentiality, transaction integrity, and the overall security posture of the application.

---

## Assessment Information

| Category | Value |
|----------|-------|
| Assessment Type | White-box |
| Target | store.bloxhouse.id |
| Assessment Period | June 2026 |
| Primary Focus | Application Security |

---

## Scope

- Invoice Generation
- Invoice Verification
- Payment Processing
- API Security
- Authorization Controls
- Business Logic
- Input Validation
- Abuse Prevention

---

## Assessment Activities

- Manual Web Application Security Testing
- API Security Assessment
- Authorization Testing
- Parameter Manipulation
- Business Logic Validation
- Replay Testing
- Input Validation Review
- Attack Surface Analysis

---

## Findings Summary

| Finding | Severity |
|---------|----------|
| Insecure Direct Object Reference (IDOR) | High |
| Public API Documentation Exposure | Medium |
| Missing Replay Protection | Medium |

---

## Security Controls Successfully Validated

The following controls were evaluated and no exploitable issues were identified during the assessment:

- Payment Bypass Protection
- Payment Amount Validation
- SQL Injection Protection
- Cross-Site Scripting (XSS) Protection
- Input Validation Controls

---

## Technologies

- Burp Suite
- Manual Security Testing
- HTTP Request Manipulation
- OWASP WSTG
- OWASP API Security Top 10

---

## Deliverables

- Executive Summary
- Technical Findings
- Risk Assessment
- Proof of Concept (PoC)
- Remediation Recommendations

---

## Key Takeaways

This engagement demonstrated practical experience in identifying authorization weaknesses, assessing API security, validating business logic implementations, and producing remediation-oriented security reports for transaction-based web applications.

---

## Disclosure

The original assessment report contains confidential implementation details.

Only high-level engagement information is presented in this repository. Sensitive information, customer data, proof-of-concept details, and internal implementation specifics have been removed.