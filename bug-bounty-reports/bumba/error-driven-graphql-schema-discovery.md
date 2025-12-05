# GraphQL Schema Exposure via Error Handling + Partial Authorization Leakage

**Type**: Research / Bug Bounty Recon
**Status**: Submitted (Closed as Informational)
**Platform**: Bug Bounty Program (via HackerOne)
**Target**: Redacted

---

## Summary

### Overview

During a GraphQL assessment on a private bug bounty program, I discovered that—even though introspection was intentionally disabled—the API still exposed significant internal schema information through error messages.

Without exploiting any functionality, I was able to:
 - Reconstruct parts of the schema through error responses
 - Identify valid fields, mutations, and input requirements
 - Observe differences between authentication vs. authorization check
 - Understand backend logic flow purely from error metadata
 - This finding was submitted responsibly and reviewed as Informational, which is expected for this class of issue.

# What I Tested

Using only safe, read-only GraphQL queries, I analyzed:

🔹 Schema behavior

 - Hidden fields revealed via “Did you mean…?” suggestions
 - Field existence inferred from “Cannot query field X” responses
 - Mutation structure revealed from required arguments

🔹 Authorization boundaries

Different responses appeared for:

 - No token
 - Invalid token
 - Valid token with insufficient privileges

These differences allowed mapping of auth layers without performing any unauthorized access.

🔹 Input object discovery

Malformed mutation attempts revealed internal object structures such as:

 - Required fields
 - Nested input types
 - Validation rules

Again - all done without executing any mutation.

## Example Observations (Redacted)

### Query:
query { users }
### Response:
Field 'users' must have a selection… Did you mean 'users { ... }'?

Revealed the users field and expected structure.

### Query:
query { users { id email role } }
### Response:
Cannot query field 'id' on type 'User'.

Exposed valid/invalid field definitions within the User type.

### Query:
mutation { updateUser }
### Response:
Argument 'user_id' of type 'String!' is required.

Disclosed mutation entry point and required argument format.

## Security Impact (Informational)

While not directly exploitable, detailed error feedback can:

 - Help attackers infer schema structure despite disabled introspection
 - Expose internal business logic and role restrictions
 - Assist in building targeted follow-up attacks (privilege escalation, logic abuse)

The issue increases reconnaissance accuracy for a potential attacker but does not, by itself, provide unauthorized access.

## Skills Demonstrated

This assessment highlights my ability to perform:

 - GraphQL schema discovery without introspection
 - Error-based enumeration techniques
 - Authentication/authorization flow analysis
 - Safe testing methodology
 - API recon in bug bounty environments
 - Writing structured and professional security documentation

## Notes

 - No mutation or state-changing operation was executed.
 - No unauthorized access was attempted.
 - Full compliance with program scope and rules.
