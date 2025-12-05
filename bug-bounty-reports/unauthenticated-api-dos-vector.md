# Unauthenticated POST Request Causing Server Stall (Potential DoS Vector)

**Status:** Submitted (pending re-evaluation from informational)
**Severity (Suggested):** Medium → High (Application-Layer DoS Pontetial)  
**Target Scope:** [Redacted] Bug Bounty Program from Hackerone

---

## Summary

During testing of publicly accessible versioned API endpoints, I identified that unauthenticated POST requests can cause significant response delays (up to ~135 seconds). The behavior is consistent and reproducible, indicating a potential Application-Layer Denial of Service (DoS) via resource exhaustion. This extends a previously reported endpoint enumeration issue into a practical exploitation scenario and demonstrates unintended processing of unvalidated payloads.

---

## Affected Endpoints

(Paths redacted)

| Endpoint | Authentication Required | Method Allowed | Result |
|----------|-------------------------|----------------|--------|
| `/3/.../.../...` | ❌ No | POST, OPTIONS | Vulnerable |
| `/1/.../.../...` | ❌ No | POST, OPTIONS | Vulnerable |

**Evidence of Allowed Methods:**

OPTIONS /1/.../.../... → 200 OK
allow: POST, OPTIONS

OPTIONS /3/.../.../...→ 200 OK
allow: POST, OPTIONS

Both endpoints accept POST requests without authentication and process large payloads, leading to backend processing stall.

---

## Impact

A remote attacker—without authentication—can trigger expensive backend operations by repeatedly sending crafted POST requests.

### Possible outcomes:

 - Backend worker/thread exhaustion

 - Severe latency spikes

 - Degraded performance across upstream services

 - Potential service disruption depending on autoscaling behavior

Because the vector requires no valid credentials, it is scalable and can be executed by any external actor.

---

##  Steps to Reproduce

### 1. Test Payload

Only safe, non-malicious random data was used during testing.

### 2. Test Command

time curl -s -o /dev/null -X POST \
"https://[redacted]/1/[redacted]/[redacted]/[redacted]" \
-H "Content-Type: application/json" \
-d '{"loop":"'$(head -c 5000 /dev/urandom | base64)'"}'

---

## Observed Results

/3/[redacted]/[redacted]/[redacted]

| Attempt | Responses Time | Behavior |
|----------|-------------------------|----------------|
| #1 | ~0.53s | Normal |
| #2 | ~0.91s | Latency Increased |
| #3 | ~135.73s | Backend Stall |

/1/[redacted]/[redacted]/[redacted]
| Attempt | Responses Time | Behavior |
|----------|-------------------------|----------------|
| #1 | ~0.64s | Normal |
| #2 | ~134.87s | Backend Stall |

The pattern is consistent across endpoints, confirming a backend bottleneck rather than network fluctuations.

---

## Analysis

The endpoints attempt to fully process arbitrary JSON payloads without:

 - Authentication

 - Input validation

 - Payload size limits

 - Rate limiting

This results in:

 - Unbounded processing time

 - No early rejection for invalid or unauthenticated requests

 - Easy exploitation at low cost

This behavior makes the endpoint vulnerable to resource exhaustion even with minimal traffic.

---

## Expected Behavior

 - Enforce authentication for POST operations

 - Apply rate limiting on unauthenticated requests

 - Reject oversized or invalid payloads

 - Implement early-return validation (“fail fast”) to prevent backend stalls

A properly configured endpoint should reject invalid POST payloads within <100ms, not attempt full processing.

---

## Reference Standards

 - CWE-400 — Uncontrolled Resource Consumption
 - CWE-770 — Allocation Without Limits
 - OWASP API4:2023 — Unrestricted Resource Consumption
   
---

 ## Environment Notes

 - Testing performed under stable network conditions (baseline ~15–16ms).

 - No load tools or automation used—single curl requests triggered stalls.

 - Only safe testing was conducted; no brute force or traffic flooding.

---
