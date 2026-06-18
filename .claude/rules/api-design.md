---
paths:
  - "src/api/**"
  - "src/**/routes/**"
  - "src/**/handlers/**"
  - "src/**/controllers/**"
---

# API design rules

> Auto-loads only when you touch an endpoint, route, or handler — that's what the `paths:` globs above do. Tune them to your layout.

Replace the examples below with your project's real conventions. Short beats complete.

## Shape

- Resources are nouns, plural: `/invoices`, `/invoices/{id}`. Actions are HTTP verbs, not URL segments.
- Request and response bodies are JSON. Field names are camelCase and stable — renaming a field is a breaking change.

## Status codes

- `200` for a successful read or update, `201` for a create, `204` for a delete with no body.
- `400` for malformed input, `401` unauthenticated, `403` authenticated-but-not-allowed, `404` not found, `409` for a conflict.
- Never return `200` with an error payload. The status code is the contract.

## Errors

- Error responses share one shape: `{ "error": { "code": "...", "message": "..." } }`.
- `message` is for humans; `code` is for clients to branch on. Don't make clients string-match the message.

## Versioning

- Breaking changes go behind a new version prefix (`/v2/...`). Additive changes don't need one.

## When in doubt

If an endpoint's behavior or contract changes, run `well-actually-documentation-recent` so the system docs stay accurate.
