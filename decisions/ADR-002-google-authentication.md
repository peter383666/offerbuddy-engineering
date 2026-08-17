# ADR-002: Use Google OIDC with a Server-Side OfferBuddy Session

## Status

Accepted and implemented.

## Date

3 August 2026; implemented architecture updated after Sprint 1.

## Context

Sprint 1 needed a low-friction identity mechanism without building password registration, storage, verification, or reset flows. The browser application also needed a secure authenticated context for user-owned application APIs.

The authentication architecture had to decide both who authenticates the user and how OfferBuddy maintains the application session.

## Decision

Use Google OAuth 2.0/OpenID Connect as the only Sprint 1 identity provider and Spring Security as the relying-party implementation.

After Google authentication:

1. Spring Security processes the OIDC callback.
2. OfferBuddy maps the verified Google subject to a local `users` row.
3. The backend establishes a server-side servlet session.
4. The browser receives a `JSESSIONID` cookie.
5. Subsequent API requests derive the user from the authenticated server context.

The React application does not store or attach Google access tokens to normal API requests.

## Session and Request Security

- Session creation policy is `IF_REQUIRED`.
- The session cookie is HTTP-only and named `JSESSIONID`.
- Production sets `Secure` and `SameSite=Lax`.
- The configured session timeout is eight hours.
- Spring Security CSRF protection remains enabled.
- A readable CSRF cookie is paired with the `X-XSRF-TOKEN` request header for state-changing API calls.
- Protected API requests without a session return JSON `401` responses instead of redirecting to Google.
- Authentication starts at `/oauth2/authorization/google`; the callback is `/login/oauth2/code/google`.
- Logout invalidates the OfferBuddy session. It does not sign the user out of Google.

Same-origin production hosting through Nginx keeps the React frontend, API, OAuth callback, session cookie, and CSRF behaviour under `https://offerbuddy.io`.

## Local Identity

The local user stores:

- public OfferBuddy UUID
- email
- display name and avatar when supplied
- authentication provider
- provider subject

A unique `(auth_provider, provider_user_id)` constraint prevents duplicate local identities for the same Google subject.

## Consequences

### Positive

- OfferBuddy does not handle passwords.
- Browser code does not manage bearer tokens.
- Server-side logout and authorisation use established Spring Security behaviour.
- Same-origin cookies simplify the deployed web application.
- User ownership is derived from the authenticated principal.

### Negative

- Google availability and OAuth configuration affect sign-in.
- Server-side sessions are local to the backend instance.
- Horizontal backend scaling would require session affinity or shared session storage.
- Cookie authentication requires correct CSRF, cookie, proxy, and HTTPS configuration.

## Alternatives Considered

### Email and password

Rejected for Sprint 1 because it adds password storage, reset, verification, and abuse-protection scope.

### Browser-managed OAuth tokens

Rejected because it would move token storage and refresh responsibility into the frontend without a Sprint 1 need for third-party API clients.

### Stateless OfferBuddy JWTs

Rejected because the single same-origin web application did not need stateless authentication, and server-side invalidation was simpler.

### Multiple identity providers

Deferred until there is a demonstrated product requirement.

## Implementation Outcome

Google OIDC, Spring Security, `JSESSIONID`, server-side sessions, and CSRF protection are running in production. Redis is not used for Sprint 1 sessions. Revisit session storage only if multiple backend instances or another concrete session-sharing requirement is introduced.
