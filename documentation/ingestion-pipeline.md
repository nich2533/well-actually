# Ingestion pipeline

> How external data enters the system and becomes queryable. System knowledge — keep it matching the code; run `/documentation` when a stage changes.

Replace the placeholder description below with your real pipeline.

## Stages

1. **Receive.** Raw payloads land via webhook or upload and are written to a staging store untouched. Nothing is parsed yet — staging is the audit trail.
2. **Validate.** Each record is checked against a schema. Failures go to a dead-letter queue with the reason attached; they are never silently dropped.
3. **Normalize.** Valid records are mapped to the internal model — units converted, timestamps moved to UTC, IDs resolved against existing entities.
4. **Persist.** Normalized records are upserted. Re-ingesting the same source record updates rather than duplicates.

## The non-obvious parts

- **Idempotency key.** Dedup is keyed on `(source, source_id)`, not the payload hash. A corrected record from the same source overwrites the original by design.
- **Ordering.** Records are not guaranteed to arrive in order. Normalize handles out-of-order updates by comparing the source's own timestamp, not arrival time.
- **Dead-letter is not a graveyard.** The DLQ is replayable. Fixing a validation bug and replaying is the intended recovery path, so DLQ entries keep the full raw payload.

## Where it lives

- Receive + staging: `src/ingest/receive`
- Validation + DLQ: `src/ingest/validate`
- Normalize + persist: `src/ingest/normalize`
