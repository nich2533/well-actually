# Testing rules

> Load this when writing or changing tests, or when "is this done?" comes up.

Replace the examples below with your project's real conventions.

## What to test

- Test behavior, not implementation. A test that breaks on a refactor that didn't change behavior is a bad test.
- Every bug fix ships with a test that fails before the fix and passes after. That's how a fix stops being a re-fix.
- Cover the boundaries: empty input, the maximum, the off-by-one, the unauthorized caller.

## How

- One assertion-worth of behavior per test. When a test fails, its name should tell you what broke.
- No network, clock, or filesystem in unit tests — inject those. Integration tests own the real I/O.
- Tests are independent and order-free. No shared mutable state between them.

## Done means

- The change has tests, the suite is green, and the new tests actually fail when you break the code they cover.

## When in doubt

If a change alters expected behavior, the docs describing that behavior need updating too — run `/documentation`.
