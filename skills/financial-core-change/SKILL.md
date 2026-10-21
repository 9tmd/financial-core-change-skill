---
name: financial-core-change
description: Safely change financial-core code in multi-tenant systems (ledgers, payments, billing). Enforces single-writer zones, checker/runtime mirror, conservation invariants, tiered review, version sync, and prod-data guards so money can never appear or disappear. Use whenever a change touches balances, payouts, migrations, reversals, or money-movement logic.
---

# Financial-Core Change Discipline

A checklist for changing the money-critical core of a multi-tenant financial system.
The highest rule is one sentence: **money can never appear or disappear.**

## When to use

Any change touching balances, payouts/settlements, money movement, migrations, reversals,
or write-protected financial tables.

## 1. Single-writer zones

The financial core, database migrations, and version numbers are single-writer zones —
only one session touches them at a time. Parallelize frontend views, reports, tests, and
read-only research; never parallelize these three.

## 2. Mirror the checker (the easiest mistake, and the most fatal)

If the runtime write path changes, the pure-function checker must mirror it, or
reconciliation drifts:

- runtime write path ↔ pure-function checker (forward/reverse pairs: apply/undo, carry/un-carry)
- replay, conservation, and recompute paths must use the same calibers as the runtime

A commit that changes only the write path and not the checker is a hard failure.

## 3. Conservation invariant

For anything touching fees, payouts, reversals, or balance boundaries, verify the
conservation identity holds (e.g. `total_fee == Σ each split`). Keep the two ledgers
distinct: the snapshot-written fee chain vs. the read-recomputed flow ledger (locked after payout).

## 4. Tiered review (mandatory before commit)

| Tier | Scope | Handling |
|---|---|---|
| Low | pure frontend style/copy, comments, docs | skip review |
| Medium | backend API/reports, general logic | review summary + diff + open questions |
| High | financial core, conservation, migrations, data-fix scripts, payout write path | review plan + diff + tests + invariant list |

Four things every review must check: ① invariant symmetry (both sides of forward/reverse
changed); ② boundary enumeration (rest/transfer/half/reversal/bust/pending combos, no gaps);
③ tests pin the bug (red→green, assertion actually catches it); ④ data-change discipline
(dry-run → human review → apply).

A second pair of eyes reviews before commit; medium/high changes are not committed without review.

## 5. Version sync (at integration time)

Update every place before committing: backend version constant (with the prefix, if any),
frontend package version (×2 in the lockfile), and the changelog. Keep patch bumps modest;
ask before minor/major bumps.

## 6. Tests and prod-data guards

- Full test suite, serial (single DB) to avoid cross-file pollution.
- Lint + frontend build for frontend changes.
- Never touch the production env file.
- Keep production data by default; destructive/cleanup/data-fix operations always
  dry-run → review → apply.
- Seed scripts only touch the designated demo tenant, never real tenants.

## 7. Don't re-litigate definitions

Financial definitions are frozen in an authoritative doc. If an implementation diverges,
the user's latest word wins, and doc + code + memory are corrected together — never
re-confirmed repeatedly.

## Standard close-out order

1. Classify the tier; for high, map the blast radius first (checker mirror points,
   conservation identity, boundaries).
2. Change code + mirror the checker + add a test (red→green).
3. Lint + full serial test suite.
4. High tier → send for review → pass before continuing.
5. Sync version everywhere (ask before minor/major bumps).
6. Commit + push (deploy triggers automatically).
