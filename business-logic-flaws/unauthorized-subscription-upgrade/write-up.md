# Unauthorized Subscription Upgrade via Token Ownership Confusion

> **Category:** Business Logic Vulnerability  
> **Impact:** Unauthorized entitlement escalation  
> **Domain:** Subscription & Billing Systems

---

## Overview

This write‑up documents a **business logic vulnerability** in a subscription upgrade workflow that can lead to **unauthorized account upgrades**.

The issue arises from missing ownership enforcement during the final subscription state transition.  
Although upgrade tokens are correctly generated and stored server‑side, the backend fails to ensure that the token is applied **only to the account that originally requested the upgrade**.

This vulnerability affects **core billing logic** and can result in silent revenue loss.

---

## Vulnerability Class

- Business Logic Flaw
- Authorization Context Confusion
- Token Ownership Violation
- Entitlement Escalation

---

## Intended Subscription Upgrade Flow

The subscription upgrade process is designed to work as follows:

1. A user registers with a default **free** subscription  
2. The user initiates an upgrade request  
3. The backend:
   - Generates a temporary upgrade token
   - Associates it with the requesting user
   - Stores the requested subscription tier
4. The user completes payment via a third‑party provider  
5. The backend validates the upgrade and updates the subscription status  

At a high level, this design appears correct and secure.

---

## Token Handling

Upgrade tokens are stored server‑side and include:

- A random, unguessable token value
- The identifier of the requesting user
- A usage state
- (Optionally) an expiration timestamp

At this stage, the system behaves as expected and no vulnerability is present.

---

## Root Cause Analysis

### Missing Ownership Enforcement at State Change

The vulnerability originates at the **final state‑changing endpoint**, responsible for applying the subscription upgrade.

Although the backend validates that an upgrade token is valid and unused, it fails to enforce **token ownership** when determining **which account** should receive the upgraded subscription.

Specifically:

- The backend accepts a client‑controlled account identifier during the confirmation phase without verifying that it matches the user associated with the upgrade token.
- The endpoint is **exposed to any origin**, allowing arbitrary users or clients to invoke it directly.

This results in a **disconnect between authorization and target identity**, making the system vulnerable to unauthorized subscription upgrades.

---

## Security Impact

If exploited, this vulnerability may allow:

- Unauthorized subscription upgrades
- Abuse of paid features
- Entitlement escalation across accounts
- Revenue loss
- Compromise of billing integrity

The issue does not require advanced exploitation techniques and can be abused through normal application workflows.

---

## Why This Vulnerability Is Realistic

This class of vulnerability commonly appears in real‑world systems because:

- Final payment confirmation endpoints are often treated as internal APIs
- Developers assume that a valid token implies a valid operation
- Trust boundaries between users and payment providers are not strictly enforced
- Client‑controlled identifiers are used during irreversible state changes

The flaw exists even when authentication and token generation are implemented correctly.

---

## Recommended Mitigations

To prevent this issue:

- Enforce token ownership during the final confirmation step
- Derive the subscription recipient exclusively from the token itself
- Avoid accepting client‑controlled account identifiers for state changes
- Restrict confirmation endpoints to trusted payment providers
- Validate request origin and signatures

---

## Key Takeaway

> Token validation alone is insufficient.  
> Ownership must be enforced **at the exact moment the subscription state changes**.

Systems can be secure at every preliminary stage and still fail catastrophically if authorization checks are missing at the final transition.

---

## Notes

This vulnerability is unrelated to:

- Injection flaws
- Weak randomness
- Authentication bypasses

It is a **pure business logic flaw** rooted in authorization design.
