---
id: resharding-troubleshooting
title: Troubleshooting Resharding
sidebar_label: Resharding
description: Advice to ensure that the node goes through resharding successfully
---

## How resharding works {#how-resharding-works}

Since protocol version 85 (nearcore 2.13.0), NEAR splits shards automatically. A split does not need a new release or a protocol upgrade vote.

- At the end of each epoch, the protocol checks the memory usage of each shard against limits in the epoch config. At most one shard is split at an epoch boundary.
- If a split is chosen at the end of epoch N, the new shard layout takes effect at the start of epoch N+2. Your node prepares in the background during epoch N+1. It splits the shard at the end of epoch N+1. It then finishes splitting the shard state on disk in the background.
- The epoch config also limits the number of shards, and requires at least two epochs between splits.

Mainnet and testnet already have 10 shards, which is the current limit. A further split needs a new protocol version that raises the limit.

## What to do as a node operator {#operators}

- Do not hardcode the number of shards or the mapping from account to shard. Shard IDs do not run from 0 to the number of shards. Read the current shard layout from the `shard_layout` field of the `EXPERIMENTAL_protocol_config` RPC method. Expect it to change at epoch boundaries. The same response also has a `dynamic_resharding` field, which is `false` on mainnet and testnet. It is a legacy genesis field and does not show whether shards are split automatically.
- Watch the `near_resharding_status` metric. It has one series per parent shard. The values are `1` scheduled, `2` splitting flat storage, `3` flat storage catch-up, `4` resharding trie state, `5` done, `-1` failed and `-2` cancelled. The metrics `near_resharding_start_timestamp_seconds` and `near_resharding_total_duration_seconds` show when a split started and how long it took.
- Avoid restarting the node while a split is in progress (`near_resharding_status` from `2` to `4`). If the node stops during a split and then refuses to start with an error that says `must run resume_resharding`, the error names the parent shard, for example `s8.v3`. Run `neard flat-storage resume-resharding --shard-id 8` with the number of that shard, and then start the node.
- If a split fails (`-1`) on a non-archival node, stop the node, remove the `data` folder in the node's home directory, and start the node again. The node then catches up with [epoch sync](/intro/node-epoch-sync).

For details, see [Dynamic Resharding](https://github.com/near/nearcore/blob/master/docs/architecture/how/dynamic_resharding.md) in the nearcore repository.

## Earlier reshardings {#history}

Before protocol version 85, each change to the shard layout came with a release and a protocol upgrade. For example, nearcore 1.37.0 split mainnet from 4 to 5 shards in March 2024. That resharding is complete, and its instructions no longer apply.
