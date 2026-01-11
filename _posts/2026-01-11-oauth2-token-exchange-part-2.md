---
layout: post
title: "OAuth 2.0 Token Exchange in Practice (Advanced)"
categories: [oauth2, iam, security, backend]
---

> This article is Part 2 of a series on OAuth 2.0 Token Exchange.
> Read Part 1 for the conceptual foundations.

This article focuses on how OAuth 2.0 Token Exchange works in real systems — from request parameters to delegation traces and response semantics.

---

## 1. Mental Model: What Token Exchange Really Is

At its core, token exchange is a request to an Authorization Server (AS) that says:

“Given this token, issue a new token that is better suited for a different target.”

The new token may differ in:

* audience (aud)
* resource binding
* scope
* lifetime
* cryptographic properties
* delegation context

The AS is not copying claims; it is evaluating policy.

---

## 2. Token Exchange Endpoint

Token exchange is performed at the OAuth 2.0 `/token` endpoint.

Important detail:

The AS performing the exchange does not need to be the same AS that issued the original token.

This enables:

* homogeneous exchanges (same trust domain)
* heterogeneous exchanges (cross trust domains)

---

## 3. Core Request Parameters

### grant_type

```
urn:ietf:params:oauth:grant-type:token-exchange
```

Explicitly indicates a token exchange request.

---

### subject_token

The token being exchanged.

Conceptually:

* defines who the new token is about
* the resulting token’s subject (sub) is derived from this token

---

### subject_token_type

Declares how the AS should validate the subject_token.

Common values:

* urn:ietf:params:oauth:token-type:access_token
* urn:ietf:params:oauth:token-type:id_token
* urn:ietf:params:oauth:token-type:refresh_token
* urn:ietf:params:oauth:token-type:saml1
* urn:ietf:params:oauth:token-type:saml2

For opaque tokens, the AS must rely on introspection or internal validation.

---

## 4. Policy-Shaping Parameters

These parameters actively influence how the issued token is constructed.

---

### resource

Identifies the concrete resource the token will be used against.

* typically an absolute URI
* enables resource-specific policy enforcement

Example policies:

* mandatory token encryption
* specific signing algorithms
* reduced token lifetime

---

### audience

Identifies the logical service that will consume the token.

* often a symbolic name (e.g. user-svc)
* known to both the client and the AS

---

### How resource and audience Combine

They are additive, not redundant.

If:

* resource requires encrypted tokens
* audience requires a specific signing algorithm

The AS may issue a token satisfying both constraints — effectively a cartesian product of policies.

---

### scope

A space-delimited list of requested scopes.

Rules:

* scope reduction is common
* scope escalation is tightly controlled
* least privilege is always enforced

Microservice example:

Service A has:

```
scope-a scope-b
```

To call Service B, it needs:

```
scope-a scope-b scope-c
```

Service A may request scope-c during exchange.
The AS decides whether it is allowed.

---

## 5. Delegation-Specific Parameters

Delegation is explicit.

---

### actor_token

Identifies who is acting on behalf of the subject.

* required for delegation
* omitted for impersonation

---

### actor_token_type

Declares the type of actor_token.

Values mirror subject_token_type.

---

## 6. Delegation Trace: act Claim

When delegation occurs, the AS records it in the act claim.

```
{
  "sub": "user@example.com",
  "act": {
    "sub": "service_B",
    "act": {
      "sub": "service_A"
    }
  }
}
```

Rules:

* the top-most actor is the most recent
* nested actors represent earlier hops
* consumers should apply policy based on the most recent actor

---

## 7. may_act Claim

The may_act claim defines who is allowed to act as a delegate in future exchanges.

```
{
  "sub": "userA@example.com",
  "act": { "sub": "service_A" },
  "may_act": { "sub": "service_B" }
}
```

This prevents uncontrolled delegation.

---

## 8. Token Exchange Response Structure (Critical)

Token exchange responses are type-agnostic containers.

---

### access_token

Always contains the issued token, regardless of its actual type.

Clients must not infer semantics from this field.

---

### issued_token_type

Indicates what kind of token was issued.

Common values:

* urn:ietf:params:oauth:token-type:access_token
* urn:ietf:params:oauth:token-type:id_token
* urn:ietf:params:oauth:token-type:saml1
* urn:ietf:params:oauth:token-type:saml2

This field is essential for correct client behavior.

---

### token_type

Describes how the token is used.

Examples:

* Bearer
* N_A

Do not confuse this with issued_token_type.

---

### expires_in (optional)

Lifetime of the issued token in seconds.

---

### scope (optional)

The authoritative scopes granted during exchange.

Clients should treat this as the source of truth.

---

### refresh_token (optional)

Rarely issued for exchanged tokens.

Most AS implementations avoid refresh tokens to limit blast radius.

---

## 9. End-to-End HTTP Flow

### Step 1: Original Resource Request

```
GET /resource HTTP/1.1
Host: frontend.example.com
Authorization: Bearer <token>
```

---

### Step 2: Token Exchange Request

```
POST /token HTTP/1.1
Host: as.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic <client-credentials>

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&resource=https://backend.example.com/api
&subject_token=<token>
&subject_token_type=urn:ietf:params:oauth:token-type:access_token
```

---

### Step 3: Delegated Exchange (Optional)

```
POST /token HTTP/1.1
Host: as.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic <client-credentials>

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&resource=https://backend.example.com/api
&subject_token=<token>
&subject_token_type=urn:ietf:params:oauth:token-type:access_token
&actor_token=<service-token>
&actor_token_type=urn:ietf:params:oauth:token-type:access_token
```

---

### Step 4: Token Exchange Response

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-cache, no-store

{
  "access_token": "<jwt>",
  "issued_token_type": "urn:ietf:params:oauth:token-type:access_token",
  "token_type": "Bearer",
  "expires_in": 60,
  "scope": "api"
}
```

---

## Final Thoughts

OAuth 2.0 Token Exchange is a foundational primitive for:

* zero-trust architectures
* secure microservice communication
* auditable delegation chains
* policy-driven authorization

Used correctly, it enables systems to scale securely without sacrificing identity or control.
