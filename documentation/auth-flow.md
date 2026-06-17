# Auth flow

> How a request becomes an authenticated, authorized caller. This is system knowledge — *what is*, not *what should be*. Keep it matching the code; run `/documentation` when the flow changes.

Replace the placeholder description below with your real flow.

## Overview

1. The client obtains a token from the auth provider (login, or a refresh exchange).
2. Every protected request carries the token in `Authorization: Bearer <token>`.
3. Middleware verifies the token's signature and expiry, then loads the caller's identity and roles onto the request context.
4. Handlers read identity from the request context. They never re-parse the token.

## The non-obvious parts

- **Clock skew.** Token expiry is checked with a small leeway to tolerate skew between services. The value lives in the middleware config — changing it changes how long an expired token is still accepted.
- **Refresh races.** Two concurrent requests can both trigger a refresh. The client dedupes in-flight refreshes; the server treats refresh as idempotent within a short window. Remove either side and you get random logouts under load.
- **Role caching.** Roles are cached per request, not per token. A role change mid-session takes effect on the next request, not instantly.

## Where it lives

- Token verification: `src/auth/middleware`
- Identity + role loading: `src/auth/identity`
- Client-side refresh dedupe: `src/client/auth`
