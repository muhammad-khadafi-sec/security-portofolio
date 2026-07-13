# 🎮 Bloxhouse

## Overview

Performed a white-box application security assessment against **bloxhouse.id**, with a primary focus on the **api.bloxhouse.id** payment and invoice processing functionality.

The engagement evaluated server-side pricing validation, payment workflow integrity, invoice generation logic, and the enforcement of trust boundaries between client-controlled input and backend financial processing.

---

## Assessment Information

| Category | Value |
|----------|-------|
| Assessment Type | White-box |
| Target | bloxhouse.id / api.bloxhouse.id |
| Assessment Period | March 2026 |
| Primary Focus | Transaction Security |

---

## Scope

- Public Web Application
- Payment API
- Invoice Generation
- Payment Workflow
- Pricing Validation
- Business Logic
- Transaction Integrity
- Server-Side Validation

---

## Assessment Activities

- Manual Web Application Security Testing
- API Request Manipulation
- Parameter Tampering
- Business Logic Validation
- Payment Bypass Testing
- Payment Amount Manipulation Testing
- Invoice Pricing Validation
- Transaction Integrity Review

---

## Findings Summary

| Finding | Severity | Status |
|---------|----------|--------|
| Improper Price Validation in Invoice Generation API | Medium | Identified |
| Payment Bypass | Informational | Not Identified |
| Direct Payment Amount Manipulation | Informational | Not Identified |

---

## Security Controls Successfully Validated

The following controls were evaluated and no exploitable bypass was identified during the assessment:

- Primary Payment Workflow Validation
- Direct Payment Amount Validation
- Backend Revalidation of Manipulated Payment Values
- Minimum Invoice Pricing Threshold

---

## Technologies & Methodologies

- Burp Suite Community Edition
- Manual Security Testing
- HTTP Request Manipulation
- Business Logic Testing
- OWASP Web Security Testing Guide
- CWE-840: Business Logic Errors
- CWE-20: Improper Input Validation

---

## Deliverables

- Executive Summary
- Technical Finding Documentation
- Risk and Impact Analysis
- Proof of Concept
- Remediation Recommendations
- Validation of Existing Security Controls

---

## Key Takeaways

This engagement demonstrated practical experience in assessing payment and invoice processing systems, validating backend financial controls, and identifying weaknesses in server-side pricing enforcement.

Although no direct payment bypass was achieved, the assessment identified a trust-boundary weakness where client-supplied invoice pricing values were processed without sufficient recalculation against authoritative backend pricing data.

---

## Disclosure

Only high-level engagement information is presented in this repository.

Sensitive implementation details, request payloads, internal endpoint information, and client-specific evidence have been removed or generalized prior to publication.