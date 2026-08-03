# ADR-002: Use Google Authentication as the Initial Authentication Method

## Status

Accepted

## Date

3 August 2026

---

## Context

OfferBuddy requires user authentication to ensure that each user can access only their own application records.

Several authentication approaches were considered for the MVP.

The product is initially intended for personal use and will later evolve into a multi-user SaaS application.

The authentication solution should:

* Minimise implementation complexity
* Provide a secure sign-in experience
* Avoid unnecessary account management features
* Support future expansion to multiple authentication providers

---

## Decision

The MVP will support **Google Sign-In** as the only authentication method.

OfferBuddy will use Google OAuth 2.0 and OpenID Connect to authenticate users.

After successful authentication:

1. Google verifies the user's identity.
2. OfferBuddy validates the returned identity information.
3. OfferBuddy creates or retrieves the local user account.
4. OfferBuddy establishes an authenticated application session.

OfferBuddy does not manage user passwords.

---

## Rationale

Google Sign-In was selected because it:

* Eliminates password storage.
* Eliminates password reset functionality.
* Eliminates email verification.
* Provides a familiar sign-in experience.
* Reduces MVP development effort.
* Uses a mature, widely adopted authentication standard.
* Supports future SaaS growth.

Since the target users are software engineers, requiring a Google account is considered acceptable for the initial release.

---

## Scope

The MVP supports:

* Google Sign-In
* Google Sign-Out
* Persistent authenticated sessions
* Local OfferBuddy user accounts linked to Google identities

The MVP does **not** support:

* Email and password login
* Password reset
* Email verification
* Microsoft login
* Apple login
* GitHub login
* Multiple linked identities

---

## User Identity Model

Google authentication identifies the user.

OfferBuddy authorises the user.

Therefore:

* Google manages user credentials.
* OfferBuddy manages application permissions and ownership.

Every authenticated user has a local OfferBuddy user record.

The local record owns:

* Applications
* Companies
* Jobs
* Parsing requests

---

## Consequences

### Positive

* Smaller authentication scope
* Faster MVP delivery
* Lower security risk related to password handling
* Better user experience
* Easier future expansion to additional providers

### Negative

* Google becomes an external dependency.
* Users without Google accounts cannot use the MVP.
* Google service outages may affect new logins.

---

## Alternatives Considered

### Email and Password

Rejected because it would require:

* Password storage
* Password hashing
* Password reset
* Email verification
* Additional security responsibilities

These features do not contribute directly to the MVP's primary value.

### Multiple Authentication Providers

Rejected because supporting several providers at launch would increase complexity without clear user benefit.

Additional providers can be introduced later if needed.

---

## Future Evolution

Future versions may support:

* Microsoft
* Apple
* GitHub
* Email and password
* Enterprise SSO

The authentication model should allow multiple external identities to be linked to a single OfferBuddy user.

---

## Related Documents

* `architecture/system-context.md`
* `architecture/container-design.md`
* `architecture/data-model.md`
* `technology/tech-stack.md`

---

## Outcome

OfferBuddy will use Google OAuth 2.0 and OpenID Connect as the sole authentication method for the MVP.

Authentication is delegated to Google, while authorization and data ownership remain the responsibility of OfferBuddy.
