# <redacted> API — Unauthenticated POST Request Leading to Server Stall (DoS Vector)

**Status:** Submitted *(pending re-evaluation)*  
**Severity (Suggested):** Medium → High (DoS Potential)  
**Target Scope:** https://<redacted>.com/

---

## Summary

An unauthenticated POST request to specific versioned API endpoints causes the backend to stall and significantly delay responses (~135 seconds). Repeating the request produces consistent delays, demonstrating a potential **Application-Layer Denial of Service (Layer 7 resource exhaustion)**.

This finding escalates the previously reported "endpoint enumeration" behavior into a practical exploit scenario.

---

## Affected Endpoints

| Endpoint | Authentication Required | Method Allowed | Result |
|----------|-------------------------|----------------|--------|
| `/3/<redacted>/<redacted>/<redacted>` | ❌ No | POST, OPTIONS | Vulnerable |
| `/1/<redacted>/<redacted>/<redacted>` | ❌ No | POST, OPTIONS | Vulnerable |

**Evidence of Allowed Methods:**

OPTIONS /1/<redacted>/<redacted>/<redacted> → 200 OK
allow: POST, OPTIONS

OPTIONS /1/<redacted>/<redacted>/<redacted> → 200 OK
allow: POST, OPTIONS

---

## Impact

A remote unauthenticated attacker can repeatedly send crafted POST requests to degrade service availability.

Possible outcomes:

- Resource exhaustion
- Thread/worker starvation
- Increased response time across upstream services
- Partial or full outage depending on scaling behavior

This attack **does not require login**, meaning it can be triggered at scale.

---

##  Steps to Reproduce

### Payload Used

### Test Command
```bash
time curl -s -o /dev/null -X POST \
"https://<redacted>.com/1/<redacted>/<redacted>/<redacted>" \
-H "Content-Type: application/json" \
-d '{"loop":"'$(head -c 5000 /dev/urandom | base64)'"}'

## Observed Results

/3/<redactec>/<redacted>/<redacted>

| Attempt | Responses Time | Behavior |
|----------|-------------------------|----------------|
| #1 | ~0.53s | Normal |
| #2 | ~0.91s | Latency Increased |
| #3 | ~135.73s | Backend Stall |

/1/<redactec>/<redacted>/<redacted>
| Attempt | Responses Time | Behavior |
|----------|-------------------------|----------------|
| #1 | ~0.64s | Normal |
| #2 | ~134.87s | Backend Stall |

## Analysis

The endpoint attempts to process payloads rather than validate or reject them, leading to unbounded processing time. Lack of:

 - authentication enforcement
 - request throttling/rate-limiting
 - payload size validation

makes this behavior exploitable at scale.

## Expected Behavior

 - Reject unauthenticated POST requests
 - Apply rate limiting
 - Enforce payload validation
 - Fail fast (<100ms) instead of attempting processing

## Reference Standards

 - CWE-400 — Uncontrolled Resource Consumption
 - CWE-770 — Allocation Without Limits
 - OWASP API4:2023 — Unrestricted Resource Consumption

 ## Environment Notes

 - Stable connection (avg response time baseline: 15–16ms)
 - No tooling, botting, or load generation — single curl request triggers stalc
