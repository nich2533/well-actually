# Billing webhooks

> How the payment provider's events drive billing state. System knowledge — keep it matching the code; run `/documentation` when handling changes.

Replace the placeholder description below with your real integration.

## Flow

1. The provider POSTs an event to the webhook endpoint.
2. The endpoint verifies the signature against the signing secret, then returns `200` immediately and processes asynchronously. Slow processing inside the request causes the provider to retry and double-fire.
3. The event is recorded by its provider event ID before any state change. Seeing an ID twice is a no-op.
4. The handler for that event type updates billing state (subscription active, payment failed, refund issued).

## The non-obvious parts

- **Signature first, always.** An unverified payload is discarded before parsing. The signing secret is environment-only — see `rules/security.md`.
- **Retries are the norm, not the exception.** The provider re-sends on any non-`200`, on timeout, and sometimes on success. Every handler must be idempotent; the event-ID ledger is what makes that true.
- **Out-of-order delivery.** A `payment.succeeded` can arrive after the `subscription.canceled` that followed it. Handlers reconcile against the provider's current object state rather than assuming event order.
- **The `200` is a receipt, not a result.** Returning `200` means "received and stored," not "processed successfully." Processing failures are retried from the stored event, not by asking the provider to resend.

## Where it lives

- Endpoint + signature check: `src/billing/webhook`
- Event ledger (idempotency): `src/billing/events`
- Per-event handlers: `src/billing/handlers`
