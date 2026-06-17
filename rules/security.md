# Security rules

> Load this when handling secrets, user input, auth, or external requests.

Replace the examples below with your project's real conventions. Keep this one honest — a stale security rule is worse than none.

## Secrets

- Secrets come from the environment, never from source. No keys, tokens, or passwords in code, config, or commit history.
- If a secret reaches a commit, rotate it. Deleting the commit is not enough — assume anything pushed is compromised.

## Input

- Treat every input from outside the process as hostile until validated: request bodies, query params, headers, webhook payloads.
- Validate at the boundary, once, against an explicit schema. Don't sprinkle ad-hoc checks downstream.
- Never reflect raw user input back into a response, a query, or a shell command.

## Authorization

- Authentication answers "who are you?"; authorization answers "are you allowed?". Check both — a valid token is not a permission.
- Default deny. A new endpoint is locked until you explicitly open it.

## Dependencies

- Before adding a dependency, check who maintains it and whether it runs install scripts. Prefer the one that does less.

## When in doubt

Flag it rather than guess. A security question answered wrong is a finding, not a footnote.
