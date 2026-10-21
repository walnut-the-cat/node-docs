---
id: run-rpc-node
title: Run an RPC Node
sidebar_label: Run a Node 🚀
sidebar_position: 2
description: How to run an RPC Node
---

The following instructions are applicable across localnet, testnet, and mainnet.

If you are looking to learn how to compile and run a NEAR RPC node natively for one of the following networks, this guide is for you.

- [`testnet`](/rpc/run-rpc-node#testnet)
- [`mainnet`](/rpc/run-rpc-node#mainnet)


<blockquote class="info">
<strong>Heads up</strong><br /><br />

Running a RPC node is very similar to running a [validator node](/validator/compile-and-run-a-node) as both types of node use the same `nearcore` release. The main difference for running a validator node is requiring `validator_key.json` to be used by the validator node to support its work of validating blocks and chunks on the network.

</blockquote>


## Prerequisites

- [Rust](https://www.rust-lang.org/). If not already installed, please [follow these instructions](https://doc.rust-lang.org/book/ch01-01-installation.html).
- [Git](https://git-scm.com/)
- Developer tools:
  - MacOS
    ```bash
    $ brew install cmake protobuf clang llvm awscli
    ```
  - Linux
    ```bash
    $ apt update
    $ apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo awscli
    ```
---

### Choosing your `nearcore` version

When building your NEAR node you will have two branch options to choose from depending on your desired use:

- `master` : _(**Experimental**)_
  - Use this if you want to play around with the latest code and experiment. This branch is not guaranteed to be in a fully working state and there is absolutely no guarantee it will be compatible with the current state of *mainnet* or *testnet*.
- [`Latest stable release`](https://github.com/near/nearcore/tags) : _(**Stable**)_
  - Use this if you want to run a NEAR node for *mainnet*. For *mainnet*, please use the latest stable release. This version is used by mainnet validators and other nodes and is fully compatible with the current state of *mainnet*.
- [`Latest release candidates`](https://github.com/near/nearcore/tags) : _(**Release Candidates**)_
  - Use this if you want to run a NEAR node for *testnet*. For *testnet*, we first release a RC version and then later make that release stable. For testnet, please run the latest RC version.


## `testnet`

### 1. Clone `nearcore` project from GitHub {#1-clone-nearcore-project-from-github}

First, clone the [`nearcore` repository](https://github.com/near/nearcore).

```bash
$ git clone https://github.com/near/nearcore
$ cd nearcore
$ git fetch origin --tags
```

Checkout to the branch you need if not `master` (default). Latest release is recommended. Please check the [releases page on GitHub](https://github.com/near/nearcore/releases).

Testnet usually runs a newer release candidate than mainnet. An older release cannot follow the testnet protocol version. The following commands check out the version that the public testnet RPC node runs. They need `jq`:

```bash
$ NEAR_RELEASE_VERSION=$(curl -s -X POST https://rpc.testnet.near.org -H "Content-Type: application/json" \
    -d '{"jsonrpc":"2.0","id":"dontcare","method":"status","params":[]}' | jq -r '.result.version.version')
$ git checkout "tags/${NEAR_RELEASE_VERSION:?}" -b mynode
```

### 2. Compile `nearcore` binary {#2-compile-nearcore-binary}

In the `nearcore` folder run the following commands:

```bash
$ make release
```

This will start the compilation process. It will take some time
depending on your machine power (e.g. i9 8-core CPU, 32 GB RAM, SSD
takes approximately 25 minutes). Note that compilation will need over
1 GB of memory per virtual core the machine has. If the build fails
with processes being killed, you might want to try reducing number of
parallel jobs, for example: `CARGO_BUILD_JOBS=8 make release`.

The binary path is `target/release/neard`

### 3. Initialize working directory {#3-initialize-working-directory}

The NEAR node requires a working directory with a couple of configuration files. Generate the initial required working directory by running:

```bash
$ ./target/release/neard --home ~/.near init --chain-id testnet --download-genesis --download-config rpc
```

> You can specify trusted boot nodes that you'd like to use by passing the `--boot-nodes` flag during `init`.
To find a set of active boot nodes, follow these instructions for [testnet](/intro/node-epoch-sync#testnet).

> You can skip the `--home` argument if you are fine with the default working directory in `~/.near`. If not, pass your preferred location.

This command will create the required directory structure and will generate `config.json`, `node_key.json`, and `genesis.json` for `testnet` network.
- `config.json` - Configuration parameters which are responsive for how the node will work. This file should contain the following fields critical for RPC nodes:
  - `"tracked_shards_config": "AllShards"` - to track all shards.
  - `"pollin_interval": 200ms` - to specify the time in ms to poll the transaction status at the node.
  - `"polling_timeout": 10s` - to specify the time in seconds for the transaction polling process to timeout at the node.
- `genesis.json` - A file with all the data the network started with at genesis. This contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain.
- `node_key.json` -  A file which contains a public and private key for the node. Also includes an optional `account_id` parameter which is required to run a validator node (not covered in this doc).
- `data/` -  A folder in which a NEAR node will write it's state.

> **Heads up**
> The genesis file for `testnet` is big (6GB +) so this command will be running for a while and no progress will be shown.


### 4. Get data backup {#4-get-data-backup}

⚠️ **FREE SNAPSHOT SERVICE BY FASTNEAR WILL BE DEPRECATGED STARTING JUNE 1ST, 2025. We strongly recommend all node operators to use [Epoch Sync](../intro/epoch_sync.md) when possible.**

The node is ready to be started. However, you must first sync up with the network. This means your node needs to download all the headers and blocks that other nodes in the network already have.

~~The latest daily snapshots are made available to the public by FastNear, and can be used to set up a validator node or RPC. 
For detailed instructions, please refer to [HERE](https://docs.fastnear.com/docs/snapshots).~~
While snapshot-based syncing was previously the recommended default, we now recommend Epoch Sync—a faster, more lightweight method that allows a node to catch up from genesis without downloading a large state snapshot.

### 5. Run the node {#5-run-the-node}
To start your node simply run the following command:

```bash
$ ./target/release/neard --home ~/.near run
```

That's all. The node is running you can see log outputs in your console. It will download a bit of missing data since the last backup was performed but it shouldn't take much time.


## `mainnet`

### 1. Clone `nearcore` project from GitHub {#1-clone-nearcore-project-from-github-1}

First, clone the [`nearcore` repository](https://github.com/near/nearcore).

```bash
$ git clone https://github.com/near/nearcore
$ cd nearcore
$ git fetch origin --tags
```

Next, checkout the release branch you need (recommended) if you will not be using the default `master` branch. Please check the [releases page on GitHub](https://github.com/near/nearcore/releases) for the latest release.

For more information on choosing between `master` and latest release branch [ [click here](/validator/compile-and-run-a-node#choosing-your-nearcore-version) ].

The following commands check out the latest stable release:

```bash
$ NEAR_RELEASE_VERSION=$(curl -s https://api.github.com/repos/near/nearcore/releases/latest | grep '"tag_name"' | cut -d '"' -f 4)
$ git checkout "tags/${NEAR_RELEASE_VERSION:?}" -b mynode
```

### 2. Compile `nearcore` binary {#2-compile-nearcore-binary-1}

In the `nearcore` folder run the following commands:

```bash
$ make release
```

This will start the compilation process. It will take some time
depending on your machine power (e.g. i9 8-core CPU, 32 GB RAM, SSD
takes approximately 25 minutes). Note that compilation will need over
1 GB of memory per virtual core the machine has. If the build fails
with processes being killed, you might want to try reducing number of
parallel jobs, for example: `CARGO_BUILD_JOBS=8 make release`.

The binary path is `target/release/neard`

### 3. Initialize working directory {#3-initialize-working-directory-1}

The NEAR node requires a working directory with a couple of configuration files. Generate the initial required working directory by running:

```bash
$ ./target/release/neard --home ~/.near init --chain-id mainnet --download-genesis --download-config rpc
```

> You can specify trusted boot nodes that you'd like to use by passing the `--boot-nodes` flag during `init`.
To find a set of active boot nodes, follow these instructions for [mainnet](/intro/node-epoch-sync#mainnet).

> You can skip the `--home` argument if you are fine with the default working directory in `~/.near`. If not, pass your preferred location.

This command will create the required directory structure by generating a `config.json`, `node_key.json`, and downloads a `genesis.json` for `mainnet`.
- `config.json` - Configuration parameters which are responsive for how the node will work. This file should contain the following fields critical for RPC nodes:
  - `"tracked_shards_config": "AllShards"` - to track all shards.
- `genesis.json` - A file with all the data the network started with at genesis. This contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain.
- `node_key.json` -  A file which contains a public and private key for the node. Also includes an optional `account_id` parameter which is required to run a validator node (not covered in this doc).
- `data/` -  A folder in which a NEAR node will write it's state.

### 4. Get data backup {#4-get-data-backup-1}

⚠️ **FREE SNAPSHOT SERVICE BY FASTNEAR WILL BE DEPRECATGED STARTING JUNE 1ST, 2025. We strongly recommend all node operators to use [Epoch Sync](../intro/epoch_sync.md) when possible.**

The node is ready to be started. However, you must first sync up with the network. This means your node needs to download all the headers and blocks that other nodes in the network already have.

~~The latest daily snapshots are made available to the public by FastNear, and can be used to set up a validator node or RPC. 
For detailed instructions, please refer to [HERE](https://docs.fastnear.com/docs/snapshots).~~
While snapshot-based syncing was previously the recommended default, we now recommend Epoch Sync—a faster, more lightweight method that allows a node to catch up from genesis without downloading a large state snapshot.

### 5. Run the node {#5-run-the-node}
To start your node simply run the following command:

```bash
$ ./target/release/neard --home ~/.near run
```

That's all. The node is running and you can see log outputs in your console. It will download a bit of missing data since the last backup was performed but it shouldn't take much time.



## Running a node in `light` mode
Running a node in `light` mode allows the operator to access chain level data, not state level data. You can also use the `light` node to submit transactions or verify certain proofs. To run a node in `light` mode that does not track any shards, set `tracked_shards_config` to `NoShards` in `config.json`. Make sure the file has no `tracked_shards` key. The node refuses to start when both keys are set.

```
"tracked_shards_config": "NoShards"
```

## Using memtrie for busy shards {#memtrie}

A node stores the state of each tracked shard in its database on disk. With memtrie (in-memory trie), the node also keeps the state trie of a shard in RAM. When a node applies a chunk, it reads and updates the state trie of that shard. With a memtrie, the node does this in memory instead of reading trie nodes from disk. So it applies the chunks of that shard faster. RPC view requests, such as `query` calls, do not use memtrie.

Memtrie uses a lot of RAM, and the amount grows with the size of the shard's state. A memtrie also makes the node take longer to start, because the node builds it at startup.

The reference RPC `config.json` keeps memtrie off: `store.load_mem_tries_for_tracked_shards` is `false`. Setting `tracked_shards_config` to `"AllShards"` does not turn memtrie on. Keep `store.load_mem_tries_for_tracked_shards` set to `false` on an RPC node that tracks all shards. If you set it to `true`, the node loads a memtrie for every shard it tracks.

### When to consider memtrie

Consider memtrie for one shard when your RPC node falls behind the chain. This happens when the node cannot apply the chunks of a busy shard fast enough. These are the signs:

- The `latest_block_height` that the `status` method returns on your node is lower than the one from a public RPC node. The gap does not close over time.
- The time to apply chunks of one shard is much higher than for the other shards.

To compare the time per shard, read the node's metrics:

```bash
curl -s http://127.0.0.1:3030/metrics | grep -E '^near_applying_chunks_time_(sum|count)\{apply_reason="update_shard"'
```

For each `shard_id`, divide the `_sum` value by the `_count` value. The result is the average time in seconds to apply one chunk of that shard, since the node started. If `rpc.prometheus_addr` is set in `config.json`, read the metrics from that address instead.

### Enable memtrie for one shard

List the shards that need a memtrie in `store.load_mem_tries_for_shards`. Each entry is a shard UID in the form `s<shard id>.v<shard layout version>`. The shard layout version is `3` on mainnet and testnet. For example, to load a memtrie only for shard 8, set these two fields in the `store` section of `config.json`:

```json
"load_mem_tries_for_tracked_shards": false,
"load_mem_tries_for_shards": ["s8.v3"]
```

Then restart the node. The node loads the memtrie at startup.

- The node only loads memtries for shards that it tracks. Entries for other shards are ignored.
- If `load_mem_tries_for_tracked_shards` is `true`, the node loads a memtrie for every tracked shard. The `load_mem_tries_for_shards` list then makes no difference.
- Shard IDs change when a shard is split. After a resharding, find the current shard IDs in the `shard_layout` field of the `EXPERIMENTAL_protocol_config` RPC method. Then update the list.
