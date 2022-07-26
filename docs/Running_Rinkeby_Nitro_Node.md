---
id: Running_Rinkeby_Nitro_Node
title: Running full Nitro node for Arbitrum Rinkeby TestNet after Nitro upgrade
sidebar_label: Running a Rinkeby Nitro Node
---

Note: This Rinkeby Nitro image will not work until after the Nitro upgrade scheduled for Thursday, July 28th.

Note: If you’re interested in accessing the Arbitrum Rinkeby network but you don’t want to setup your own node, see our [Node Providers](https://developer.offchainlabs.com/docs/node_providers) to get RPC access to fully-managed nodes hosted by one of our partners!

### Required Artifacts

- Latest Docker Image: `offchainlabs/nitro-node:v2.0.0-beta.5-8e0bfbc`

- Rinkeby Nitro Seed Database Snapshot
  - On Thursday, July 28th the Rinkeby chain will be temporarily offline (approx. 2-4 hours), and Offchain Labs will run through a series of steps to upgrade the Arbitrum Rinkeby testnet to Arbitrum Nitro Rinkeby testnet. During that time, Offchain Labs will convert the Arbitrum Classic database to an Arbitrum Nitro database
  - The URL to download the seed database will be announced on Discord and placed on this webpage
  - If running more than one node, easiest to manually download image and host it locally for your nodes
  - Use the parameter `--init.url` to provide the URL to download the Rinkeby seed database from

### Required parameter

- `--l1.url=<Layer 1 Ethereum RPC URL>`
  - Must provide standard Rinkeby Testnet node RPC endpoint.
- `--l2.chain-id=421611`
  - Used to select Rinkeby Nitro Rollup Testnet

### Important ports

- RPC: `8547`
- WebSocket: `8548`
- Sequencer Feed: `9642`

### Putting it all together

- When running docker image, an external volume should be mounted to persist the database across restarts. The mount point should be `/home/user/.arbitrum/rinkeby-nitro`.
- Here is an example of how to run nitro-node for Rinkeby:

  ```
  docker run --rm -it  -v /some/local/dir/rinkeby-nitro/:/home/user/.arbitrum/rinkeby-nitro -p 0.0.0.0:8547:8547 -p 0.0.0.0:8548:8548 offchainlabs/nitro-node:v2.0.0-beta.5-8e0bfbc --l1.url https://l1-rinkeby-node:8545 --l2.chain-id=421611
  ```

  - Note that if you are running L1 node on localhost, you may need to add `--network host` right after `docker run` to use docker host-based networking

### Note on permissions

- The Docker image is configured to run as non-root UID 1000. This means if you are running in Linux or OSX and you are getting permission errors when trying to run the docker image, run this command to allow all users to update the persistent folders
  ```
  mkdir /some/local/dir/rinkeby-nitro
  chmod -fR 777 /some/local/dir/rinkeby-nitro
  ```

### Optional parameters

- `--node.rpc.classic-redirect=<classic node RPC`
  - If set, will redirect archive requests for pre-nitro blocks to the designated RPC, which should be a Arbitrum Classic node with archive database
- `--init.url`
  - URL to download seed database from. Only needed when starting without database
- `--http.api`
  - APIs offered over the HTTP-RPC interface (default `net,web3,eth`)
  - Add `debug` to enable tracing
- `--http.corsdomain`
  - Comma separated list of domains from which to accept cross origin requests (browser enforced)
- `--http.vhosts`
  - Comma separated list of virtual hostnames from which to accept requests (server enforced). Accepts `*` wildcard (default `localhost`)
- `--node.archive`
  - Retain past block state
- `--node.feed.input.url=<feed address>`
  - Defaults to `wss://rinkeby.arbitrum.io/feed`. If running more than a couple nodes, you will want to provide one feed relay per datacenter, see further instructions below.
- `--node.forwarding-target=<sequencer RPC>`
  - Defaults to `https://rinkeby.arbitrum.io/rpc`
- `--node.rpc.evm-timeout`
  - Defaults to `5s`, timeout used for `eth_call` (0 == no timeout)
- `--node.rpc.gas-cap`
  - Defaults to `50000000`, cap on computation gas that can be used in `eth_call`/`estimateGas` (0 = no cap)
- `--node.rpc.tx-fee-cap`
  - Defaults to `1`, cap on transaction fee (in ether) that can be sent via the RPC APIs (0 = no cap)

### Arb-Relay

- When running more than one node, you want to run a single arb-relay which can provide a feed for all your nodes.
  The arb-relay is in the same docker image.
- Here is an example of how to run nitro-relay for Rinkeby:
  ```
  docker run --rm -it  -p 0.0.0.0:9642:9642 --entrypoint relay offchainlabs/nitro-node:v2.0.0-beta.5-8e0bfbc --node.feed.input.url wss://rinkeby.arbitrum.io/feed --l2.chain-id=421611
  ```
- Here is an example of how to run nitro-node for Rinkeby with custom relay:
  ```
  docker run --rm -it  -p 0.0.0.0:8547:8547 -p 0.0.0.0:8548:8548 offchainlabs/nitro-node:v2.0.0-beta.5-8e0bfbc --l1.url https://l1-goeri-node:8545 --feed.input.url ws://local-relay-address:9642 --l2.chain-id=421611
  ```