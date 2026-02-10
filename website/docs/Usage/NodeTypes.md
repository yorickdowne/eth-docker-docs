---
title: Archive, Full or Expired node
sidebar_position: 2
sidebar_label: History and Pruning
---

## Types of nodes

You can run an Archive node, with all history and full lookup for all historical transactions. This may take large amounts of space, depending on the execution layer client.

You can run a Full node, with all history and limited lookup for historical transactions. This fits into a 4TB drive and is often the choice for RPC nodes.

You can run an Expired node, with pre-merge history and receipts gone. This fits into a 2TB drive and is often the choice for validator nodes. This is the default.

This is controlled by variables in `.env`, which can be set with `nano .env`. Switching from one type to another often requires a full resync.

`EL_NODE_TYPE` can be one of:
- `archive`
- `full`
- `pre-merge-expiry` - the default
- `pre-cancun-expiry` - supported by some EL clients, see `default.env`
- `rolling-expiry` - keeps one year of history by default. Supported by some EL clients, see `default.env`
- `aggressive-expiry` - keep minimal history. Supported by some EL clients, see `default.env`

Please be careful with expiry options: Expired history also means expired receipts, which can throw protocols such as RocketPool, SSV, Stakewise for a loop. All of these work with pre-merge-expiry, but will likely break with the other expiry options, unless they get receipts from another (unexpired) RPC endpoint.

`CL_NODE_TYPE` can be one of:
- `archive`
- `full`
- `pruned` - the default
- `pruned-with-zkproofs` - **highly** experimental running mode without an execution layer client, currently only supported by a special build of Lighthouse

## Switch from Full to Expired node

You can use `./ethd prune-history` to switch the execution layer client to history expiry, in some cases without resync. Note that Besu requires 200 GiB free for this, and
it will take less space if instead it is resynced with `./ethd resync-execution` while `EL_MINIMAL_NODE=true`

## State pruning

In addition to history, execution layer clients also carry state. Historically, the size of the state DB has been growing and can be pruned periodically.
Today, only Nethermind still requires a state prune. Note this is entirely separate from history expiry. It applies to Full and Expired nodes. Archive
nodes should never have their state pruned.

### Automatic Nethermind prune

By default, Nethermind will prune when free disk space falls below 350 GiB on mainnet, or 50 GiB on testnet. If you
want to disable that, `nano .env` and change `AUTOPRUNE_NM` to `false`.

If you have disabled automatic prune, you can run `./ethd prune-nethermind`. It will check prerequisites, online prune Nethermind, and restart it.

### Continuous Besu prune

Besu continuously prunes with BONSAI, and from 24.1.0 on also prunes its trie-logs. A long-running Besu may benefit
from a manual trie-log prune, once.

If you have a large amount of trie logs, run `./ethd prune-besu` on a  long-running Besu. It will check prerequisites, offline prune Besu trie-logs, and
restart it.

### Continuous Geth prune

Geth continuously prunes if synced with PBSS. If you are using an old hash-synced Geth, run `./ethd resync-execution`
to use PBSS. This will cause downtime while Geth syncs, which can take 6-12 hours.
