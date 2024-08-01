# Sequoia Config

Reference Config:

```yaml
######################
### Sequoia Config ###
######################

queue_interval: 10
proof_interval: 120
stray_manager:
    check_interval: 30
    refresh_interval: 120
    hands: 2
chain_config:
    bech32_prefix: jkl
    rpc_addr: http://localhost:26657
    grpc_addr: 127.0.0.1:9090
    gas_price: 0.02ujkl
    gas_adjustment: 1.5
domain: https://example.com
total_bytes_offered: 1092616192
data_directory: $HOME/.sequoia/data
api_config:
    port: 3333
    ipfs_port: 4005
    ipfs_domain: dns4/ipfs.example.com/tcp/4001
proof_threads: 1000

######################
```

## Details
### General
* `queue_interval`: How many seconds between the queue being flushed (should not be less than a single blocks duration).
* `proof_interval`: How many seconds between when then the provider scans itself for posting proofs.
* `domain`: The domain name that users will be able to access this provider from.
* `total_bytes_offered`: The amount (in bytes) of storage this machine can provide.
* `data_directory`: Where the raw file data & tree caching will be stored.
* `proof_threads`: How many files can be proven in parallel. (lower this on weak machines)
### `stray_manager`
* `check_interval`: How many seconds between checking on chain for newly claimable files.
* `refresh_interval`: How many seconds between the internally cached stray list is refreshed.
* `hands`: How many workers are searching for and claiming strays at a time.
### `chain_config`
* `bech32_prefix`: The bech32 address prefix, this is only changed for working with Jackal forks.
* `rpc_addr`: The RPC node that this provider will post TXs to & query from.
* `grpc_addr`: The GRPC node that this provider will post TXs to & query from.
* `gas_price`: The cost of gas on the network.
* `gas_adjustment`: The gas multiplier, only raise this if your TXs are often failing due to being out of gas.
### `api_config`
* `port`: The port the provider will be available on.
* `ipfs_port`: The port IPFS will use to connect with peers.
* `ipfs_domain`: Optional way to specify a domain instead of a raw port for IPFS connectivity.