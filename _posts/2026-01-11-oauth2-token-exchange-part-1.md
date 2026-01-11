---
layout: post
title: "OAuth 2.0 Token Exchange Explained"
categories: [oauth2, iam, security]
---

## Introduction

OAuth 2.0 Token Exchange is a mechanism that allows one token to be exchanged for another token that is better suited for a specific purpose.

At a high level, it answers questions like:

- How can a service securely call another service?
- How can privileges be reduced when moving across boundaries?
- How can one identity act on behalf of another in a controlled way?

Instead of reusing the same token everywhere, token exchange enables systems to issue **purpose-built tokens** with the correct audience, scope, and permissions.

---

## Why Token Exchange Exists

In real systems, a single token is rarely sufficient.

Common scenarios include:

- A frontend calling a backend service
- A backend calling another internal service
- A service needing fewer privileges for a specific operation
- Identity needing to flow across trust boundaries

Reusing the original token often fails due to:

- Audience (`aud`) mismatches
- Over-privileged scopes
- Cross-domain trust constraints

Token exchange solves this by allowing a trusted authority to mint a **new token** tailored for the next hop.

---

## What Can Be Exchanged?

Most identity-bearing tokens can participate in an exchange:

- Access tokens
- ID tokens
- Refresh tokens
- SAML assertions

The exchanged token may differ in:

- Audience
- Scope
- Resource
- Lifetime
- Cryptographic properties

---

## Impersonation vs Delegation

Token exchange supports two distinct models.

Understanding this distinction is critical.

---

### Impersonation

In impersonation, the requester asks for a token that fully represents another identity.

- The issued token is indistinguishable from the original subject’s token
- The resource server cannot tell who initiated the request
- Powerful, but risky if misused

**Example:**  
An administrator accesses a user’s account *as if they were that user*.

---

### Delegation

In delegation, the requester acts **on behalf of** another identity.

- The subject remains the same
- The acting entity is recorded
- Downstream services can apply policy based on the actor

This model preserves accountability.

---

### Simple Analogy

Think of a token as an entry badge.

- **Impersonation**: Someone else uses your badge.
- **Delegation**: A temporary badge is issued saying *“Authorized by you.”*

Both allow access — only delegation preserves traceability.

---

## When Token Exchange Is Used

Token exchange is commonly used when:

- Services call other services
- Privileges must be constrained
- Identity flows across boundaries
- Zero-trust principles are applied

It is especially common in **microservice architectures**.

---

## What’s Next?

This article focused on *why* token exchange exists and *how it behaves conceptually*.

In **Part 2**, we’ll dive into:

- The `/token` exchange request
- Request parameters
- Delegation mechanics
- Token response structure
- End-to-end HTTP flows

👉 Continue with **Part 2: OAuth 2.0 Token Exchange in Practice**
