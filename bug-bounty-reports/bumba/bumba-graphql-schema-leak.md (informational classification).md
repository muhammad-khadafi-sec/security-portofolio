# Bumba — GraphQL Schema Exposure via Error Handling + Partial Authorization Leakage

**Status:** Submitted *(Closed as Informational)*  
**Severity (Suggested):** Low/Medium  
**Target Scope:** [redacted].bumba.[redacted]/graphql

---

## Summary

Although GraphQL introspection is disabled, the API still returns highly detailed error messages. These responses leak schema structure, valid field names, mutation paths, and authorization behavior — enabling an attacker to enumerate the backend GraphQL schema without introspection.

This misconfiguration increases the attack surface and can help in crafting privilege escalation payloads or unauthorized access attempts.

---

## Affected Component

---

## How the Vulnerability Was Discovered

Initial queries were sent without introspection. Even though typical introspection was blocked, error messages leaked details such as:

- Valid object names
- Mutation names
- Required arguments
- Fields that exist but are restricted
- Authorization flow differences

---

## Steps to Reproduce

### Schema Enumeration Attempt

**Query:**

query {
  users
}

Response:
"Field 'users' of type '[User!]!' must have a selection of subfields. Did you mean 'users { ... }'?"
➡️ Confirms internal users field exists, even though it shouldn't be discoverable.

### Field Discovery via Failing Queries
query {
	users {
		id
		email
		role
	}
}

Response:
"Cannot query field 'id' on type 'User'."
➡️ The server leaks internal schema structure without introspection.

### Mutations Enumeration
mutation {
	updateUser
}

Response:
"Field 'updateUser' argument 'user_id' of type 'String!' is required."
➡️ Reveals required arguments and mutation naming conventions.

Further fuzzing revealed internal mutation candidates including:

 - update_vault
 - create_user
 - delete_user

### Authorization Behavior Leakage
| **Condition** | **API Responses** |
| no token | AUTH_GUARD_NO_JWT_FOUND_IN_HEADERS |
| invalid token | AUTH_GUARD_JWT_TOKEN_NOT_AUTHORIZED |
| valid token | Cannot query field ... (schema leak) |
➡️ This allows logical mapping of authorization layers.

### Impact

Although no direct exploitation was performed, this behavior enables:

 - Schema reconstruction without introspection
 - Role-based access discovery
 - Location of privileged mutations
 - Attack-chain preparation for:
     - privilege escalation
     - business logic abuse
     - future endpoint abuse
     - forced mutation attempts

This creates a high-fidelity recon vector, increasing the likelihood of exploitation later.

### Expected Behavior

 - Error responses should be uniform and generic.
 - Mutations and internal schema should not be discoverable without authorization.
 - Suggestions like "Did you mean…?" should be disabled.

**Example secure response:**
{"error": "Invalid request"}

### References
 - OWASP GraphQL Security Guide
 - OWASP API3 — Excessive Data Exposure
 - CWE-209 — Information Exposure Through Error Messages

### Environment Notes
 - Tested with authenticated & unauthenticated traffic
 - No exploitation or mutation execution performed
 - Only passive enumeration and error analysis used
