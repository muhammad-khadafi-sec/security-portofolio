# Guest Basket Modification via GraphQL

**Category:** Broken Access Control / IDOR  
**Platform:** Bugcrowd
**Authentication:** Not Required  
**Status:** Duplicate  

## Submission Proof

![Sanitized Submission Proof](./submission-proof-bb-access-control-01.jpg)

## Summary

A broken access control issue was identified in the GraphQL
`addProductsToBasket` mutation affecting guest shopping baskets.

The `basketId` parameter was not sufficiently bound to the
originating guest session, allowing one guest session to reference
and modify another guest session's basket.

## Impact

- Unauthorized modification of another guest user's basket
- Cross-session object access
- Potential basket and transaction-state manipulation
- Loss of integrity of guest shopping cart data

## Affected Component

- GraphQL Mutation: `addProductsToBasket`
- Guest basket functionality

## Reproduction Overview

The issue was validated using two independent guest sessions.

```text
Guest Session A
      |
      +---- Basket A

Guest Session B
      |
      +---- Basket B

Guest Session A
      |
      +---- Basket B identifier
      |
      v
addProductsToBasket
      |
      v
Basket B modified

## Technical Observation

The server accepted a basket identifier associated with a different
guest session without sufficiently validating basket ownership.

This indicates insufficient object-level authorization on the affected
basket operation.

## Expected Behavior

A guest session should only be able to modify the basket associated
with its own session.

Requests referencing a basket belonging to another session should
be rejected.

## Validation

The behavior was reproduced using two separate guest sessions.

A video PoC demonstrating the cross-session basket modification was
submitted to the affected bug bounty program.

## Testing Notes

- Testing was performed using attacker-controlled guest sessions.
- No real customer data was accessed.
- No destructive actions were performed.