# Joining a Network

## Joining Testnet

After installing `canined`. You can join the testnet by following these steps:

```
canined init <alias> --chain-id=<chain-id>
```

{% hint style="info" %}
`chain-id` for testnet is currently `lupulella-2`.
{% endhint %}

Then we want to replace our generated genesis file with the one used to start the network. We also need to set our peers and seeds.

For an updated list of peers & seeds, please check [this page](https://github.com/JackalLabs/jackal-chain-assets/blob/main/testnet/seeds.md).

```
wget -O ~/.canine/config/genesis.json https://raw.githubusercontent.com/JackalLabs/jackal-chain-assets/main/testnet/genesis.json

export SEEDS="84f520678ef59ea02f942fa6323ec562ca5a3249@45.79.161.178:26656,cecc087977336da1e9ccd2c50097cd9e7d5e1874@141.95.33.39:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" ~/.canine/config/config.toml
```

As a validator, you'll need to set a minimum gas price like so:

```
GAS="0.002ujkl"
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"$GAS\"/" $HOME/.canine/config/app.toml
```

## Joining Mainnet

{% hint style="info" %}
Perform the following as the `jackal` user.
{% endhint %}

After installing `canined`. You can join the mainnet by following these steps:

```
canined init "NODE_NAME" --chain-id=jackal-1
```

Then we want to replace our generated genesis file with the one used to start the network.

```
wget -O ~/.canine/config/genesis.json https://cdn.discordapp.com/attachments/1002389406650466405/1034968352591986859/updated_genesis2.json

SEEDS=$(wget https://raw.githubusercontent.com/JackalLabs/canine-mainnet-genesis/master/genesis/seeds.txt -q -O -)
PEERS=`curl -sL https://raw.githubusercontent.com/JackalLabs/canine-mainnet-genesis/master/genesis/peers.txt | sort -R | head -n $PEERCOUNT | awk '{print $1}' | paste -s -d, -`
GAS="0.002ujkl"

sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.canine/config/config.toml
```

### Backing up key files

The created `node_key.json` and `priv_validator_key.json` cannot be recovered. These files _**must**_ be backed up.

```
mkdir ~/key_backup
cp ~/.canine/config/node_key.json ~/key_backup
cp ~/.canine/config/priv_validator_key.json ~/key_backup
```

You should also keep an offline backup. Using a program like `WinSCP`, you can easily copy these files to your personal desktop for safe storage/backup.

### Syncing to Current Heightid="syncing-to-current-height"></a>

#### Snapshot method

Get a snapshot [here](https://polkachu.com/tendermint\_snapshots/jackal).

For the sake of this guide, the snapshot we download is named `jackal.tar.lz4`

If you plan on becoming a validator, before using the `unsafe-reset-all` flag, always besure to back up your `priv_validator_state.json` file.

```
canined unsafe-reset-all --keep-addr-book
lz4 -c -d jackal.tar.lz4  | tar -x -C $HOME/.canine
```

Then start the chain again.

#### State Sync Method

There are a couple of ways to go about doing state sync. First is the easier route. Visit [Ping.pub](https://ping.pub/jackal/statesync) for Jackals State Sync configuration settings.

Next, copy these settings from Ping.pub to your `config.toml` in the `[statesync]` section.

It should look similar to this:

```
#######################################################
###         State Sync Configuration Options        ###
#######################################################
[statesync]
# State sync rapidly bootstraps a new node by discovering, fetching, and restoring a state machine
# snapshot from peers instead of fetching and replaying historical blocks. Requires some peers in
# the network to take and serve state machine snapshots. State sync is not attempted if the node
# has any local state (LastBlockHeight > 0). The node will have a truncated block history,
# starting from the height of the snapshot.
enable = true

# RPC servers (comma-separated) for light client verification of the synced state machine and
# retrieval of state data for node bootstrapping. Also needs a trusted height and corresponding
# header hash obtained from a trusted source, and a period during which validators can be trusted.
#
# For Cosmos SDK-based chains, trust_period should usually be about 2/3 of the unbonding time (~2
# weeks) during which they can be financially punished (slashed) for misbehavior.
rpc_servers = "https://rpc.jackalprotocol.com:443,https://rpc.jackalprotocol.com:443"
trust_height = 333000
trust_hash = "1685850c2d115a86af9059bd3f36a4fbbb0e8ba7f37863d517b6d2f54116daca"
trust_period = "168h"  # 2/3 of unbonding time

# Time to spend discovering snapshots before initiating a restore.
discovery_time = "15s"

# Temporary directory for state sync snapshot chunks, defaults to the OS tempdir (typically /tmp).
# Will create a new, randomly named directory within, and remove it when done.
temp_dir = ""

# The timeout duration before re-requesting a chunk, possibly from a different
# peer (default: 1 minute).
chunk_request_timeout = "10s"

# The number of concurrent chunk fetchers to run (default: 1).
chunk_fetchers = "42"
```

State syncing can take up to a few minutes to complete. Watch the logs to ensure it's happening. When a snapshot is found, you will see output in your log that is similar to this:

```
1PM INF Discovered new snapshot format=1 hash="S.�h�F���\"\x1d6+\x1e���ޅ��`v@�ц�����" height=1810000 module=statesync
```

It will download, verify, and apply chuncks of blockchain data. When it finishes you will see it catching up to blocks

#### State Sync Method 2

The follow commandline code will edit your `config.toml` with the proper information for state syncing to the most recent snapshot 3000 blocks and beyond.

```
STATE_SYNC_RPC=https://rpc.jackalprotocol.com:443
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 3000))
SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -e "s|^enable *=.*|enable = true|" $HOME/.canine/config/config.toml
sed -i.bak -e "s|^rpc_servers *=.*|rpc_servers = \"$STATE_SYNC_RPC,$STATE_SYNC_RPC\"|" \
  $HOME/.canine/config/config.toml
sed -i.bak -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" \
  $HOME/.canine/config/config.toml
sed -i.bak -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" \
  $HOME/.canine/config/config.toml
```

When you state sync, you can start with the latest version of `canined`.

#### Versions for Sync

| block height | canined version |
| ------------ | --------------- |
| 45381        | 1.1.2           |
| 0            | 1.1.0           |

### Starting the daemon

Start the daemon and sync to the current height.

```
sudo systemctl start jackal
sudo journalctl -u jackal -f
```

Watch the logs and ensure you are either state syncing correctly, or are syncing up to the current height.
