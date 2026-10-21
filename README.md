# financial-core-change

A Claude Code skill — a discipline checklist for **safely changing the money-critical core** of multi-tenant financial systems (ledgers, payments, billing, payouts, reconciliation).

The whole skill reduces to one rule:

> **Money can never appear or disappear.**

## What it enforces

When a change touches balances, payouts, money movement, migrations, or reversals, this skill makes sure you follow a fixed discipline before committing:

1. **Single-writer zones** — financial core, migrations, and version numbers are never edited in parallel.
2. **Checker/runtime mirror** — if the runtime write path changes, the pure-function checker must mirror it (apply/undo, carry/un-carry in pairs), or reconciliation drifts.
3. **Conservation invariant** — verify the money identity holds (`total_fee == Σ each split`) and keep the two ledgers (snapshot vs. recomputed flow) distinct.
4. **Tiered review** — low/medium/high tiers, with a mandatory second pair of eyes on medium/high changes and four things every review must check (invariant symmetry, boundary enumeration, red→green tests, dry-run→review→apply).
5. **Version sync** — every versioned place updated together.
6. **Prod-data guards** — serial tests, never touch prod env, destructive operations dry-run first, seeds only touch the demo tenant.
7. **Frozen definitions** — financial definitions live in one authoritative doc; conflicts are resolved by the user's latest word and corrected in doc + code + memory together.

## Why this exists

Financial bugs are the ones you can't ship twice. This checklist encodes the hard-won discipline of a production multi-tenant financial system: the two failure modes that actually lose money are (a) changing only the runtime path and forgetting the checker, and (b) committing a money-touching change without an independent pair of eyes. Every rule here exists to catch one of those.

## Install

Via the [skills CLI](https://skills.sh):

```bash
npx skills add financial-core-change
```

Or copy it manually into your project or user skills directory:

```bash
# project-level (committed to the repo)
mkdir -p .claude/skills/financial-core-change
cp skills/financial-core-change/SKILL.md .claude/skills/financial-core-change/SKILL.md

# user-level (all projects)
mkdir -p ~/.claude/skills/financial-core-change
cp skills/financial-core-change/SKILL.md ~/.claude/skills/financial-core-change/SKILL.md
```

## Usage

The skill self-triggers on keywords like *payout, migration, reversal, balance, money-movement, financial core*. You can also invoke it explicitly:

```
/financial-core-change
```

## License

MIT — adapt it to your own financial system's specifics (file names, review session, versioning scheme).
