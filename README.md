# double-entry-ledger

A small double-entry ledger service: accounts, balanced transfers, exactly-once writes and reconciliation. Every money movement is recorded as entries that sum to zero, and a retried request never moves money twice.

> **Status: planned, not implemented.** This repository holds the plan only. Every section describes what the first release must do and the bar it must meet.

## Why

Payments, custody and exchange back offices all rest on the same core: a journal of balanced entries, writes that are safe to retry, and a regular check that internal balances match the outside world. This project builds that core small enough to read end to end, and proves its guarantees with tests rather than claiming them.

## Planned scope for version 1

| Piece | What it does |
|---|---|
| Accounts | Accounts with a currency and a type (asset, liability, equity, income, expense) |
| Transfers | Transactions of two or more entries that must sum to zero per currency |
| Exactly-once writes | An idempotency key on every write endpoint, so a retried request has exactly one effect and returns the original result |
| Pending and posted | Two-phase transfers: hold funds, then post or void |
| Balances | Current and historical balances, computed from the journal |
| Reconciliation | A job that compares ledger balances with an external source and reports any drift. The first external source is blockchain balances, from [digital-asset-backoffice](../digital-asset-backoffice); bank statements come later |
| Audit trail | An append-only record of who moved what, when and why |

### Non-goals

- Currency conversion, interest, fees and tax logic.
- A user interface beyond an API and a small read-only dashboard.

## Design guidelines

- **Money is never a floating-point number.** Amounts are integer minor units with an explicit currency.
- **The database enforces the rules.** Constraints guarantee that every transaction balances; application code is not trusted to remember.
- **The journal is append-only.** Corrections are new entries that reverse old ones, never in-place edits.
- **Concurrency is designed, not hoped for.** A design note states the transaction isolation level, the locking strategy and the retry policy.

## Planned stack

TypeScript, NestJS and PostgreSQL in a monorepo managed with Nx, with integration tests against a real PostgreSQL in a container.

## Quality bar for the first release

- **One effect per request:** a test fires the same request concurrently many times and proves exactly one effect.
- **Property-based tests:** for any sequence of transfers, the sum of all balances stays zero per currency.
- **Integration tests** against a real database, not mocks.
- **Continuous integration** runs lint, type checking and the full test suite on every push.
- **A design note** covering the data model, isolation level, idempotency storage and known limits.
- **One command to run it locally**, plus a recorded demo in this README.

## Planned milestones

1. Data model, constraints and migrations.
2. Transfers with idempotency keys and the concurrency test.
3. Pending and posted transfers.
4. Balances and the audit trail.
5. Reconciliation against an external balance source.
6. Design note, demo and first release.

## References

- [TigerBeetle](https://github.com/tigerbeetle/tigerbeetle): financial transactions database, used as a design reference.
- [Medici](https://github.com/flash-oss/medici): double-entry library for Node.js on MongoDB.

## Licence

MIT
