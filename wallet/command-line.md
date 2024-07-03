# Command Line

The Command Line Interface (CLI) is a tool for both users and developers to interact with the Jackal Protocol without a traditional GUI.

### Download and Install[​](https://docs.jackalprotocol.com/docs/using-jackal/cli/cli#download-and-install) <a href="#download-and-install" id="download-and-install"></a>

Head to the [Releases](https://github.com/JackalLabs/canine-chain/releases) page and download the binary for your system.

### Setting up an Account[​](https://docs.jackalprotocol.com/docs/using-jackal/cli/cli#setting-up-an-account) <a href="#setting-up-an-account" id="setting-up-an-account"></a>

You can create a new account like this:

```
canined keys add {account name}
```

Or you can recover an account with a seed phrase like this:

```
canined keys add {account name} --recover
```

### Buying Storage[​](https://docs.jackalprotocol.com/docs/using-jackal/cli/cli#buying-storage) <a href="#buying-storage" id="buying-storage"></a>

Buying 1TB for a single month:

```
canined tx storage buy-storage $(canined keys show {account name} -a) 31 1099511627776 ujkl --from {account name} --gas-prices=0.02ujkl
```

In this case, `31` is 31 days, or one month, you can specify the days you wish to buy storage for here. `1099511627776` is how many bytes you wish to purchase, this value is 1TiB, you can increase this or decrease this as you please.
