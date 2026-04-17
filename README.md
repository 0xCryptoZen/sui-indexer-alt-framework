# sui-indexer-alt-framework (Cetus fork)

Single-crate fork of the `sui-indexer-alt-framework` crate from
[MystenLabs/sui](https://github.com/MystenLabs/sui).

## Why a fork?

We need the ability to land local patches against the framework crate without
forking the entire `MystenLabs/sui` monorepo. By extracting just this one crate
into its own repository, we can:

- carry small Cetus-specific changes (e.g. ingestion behaviour, metrics, retries)
  on top of upstream;
- override only this crate via `[patch.crates-io]` (or a path patch) inside the
  consumer workspace (chainflow-indexer);
- continue to consume all sister crates (`sui-types`, `sui-rpc`, `sui-rpc-api`,
  `sui-indexer-alt-framework-store-traits`, `sui-indexer-alt-metrics`,
  `sui-storage`, `sui-field-count`, `sui-futures`) directly from the upstream
  `MystenLabs/sui` repo at the same pinned rev — they are NOT vendored here.

## Pinned upstream rev

`33ef98b2337036b06f9da3927715a411dcddfc40` (mainnet-v1.69.2)

The `Cargo.toml` in this repo pins every sister Sui crate to that exact rev.

## How chainflow-indexer consumes it

In `chainflow-indexer/Cargo.toml` we use a path-based patch:

```toml
[patch."https://github.com/MystenLabs/sui"]
sui-indexer-alt-framework = { path = "/home/bond/github/cetus/sui-indexer-alt-framework-fork" }
```

(or, once pushed, swap `path = ...` for `git = "..."`, `rev = "..."`.)

Only this single crate is patched — every other Sui crate keeps coming from
upstream.

## Updating from upstream

When a new upstream rev is needed:

1. Bump the rev in this repo's `Cargo.toml` for every `https://github.com/MystenLabs/sui`
   git dependency.
2. Re-sync the `src/` tree from upstream (`crates/sui-indexer-alt-framework/src`)
   at the new rev.
3. Re-apply local Cetus patches.
4. Bump the same rev in `chainflow-indexer/Cargo.toml` for the sister Sui crates
   (those still come straight from upstream).
