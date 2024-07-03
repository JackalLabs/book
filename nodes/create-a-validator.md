# Create a Validator

{% hint style="info" %}
This guide assumes you are using the same machine as the full node.

Perform the following steps as your `jackal` user.
{% endhint %}

### Creating A Wallet[​](https://docs.jackalprotocol.com/docs/nodes/validators/joining#creating-a-wallet) <a href="#creating-a-wallet" id="creating-a-wallet"></a>

We need to create a wallet and set the keyring password.

```
canined keys add WALLET_NAME --keyring-backend os
```

This wallet is used to claim rewards, commission and to vote as your validator.

You will see a similar output once created.

```
- name: WALLET_NAME
  type: local
  address: jkl1hjhglrzggqtdhsh3ag8jp0cckmva5pe976jxel
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"Rnrlv1TNrt1cz3+pSq2UDNiJQZINNlgtkNousVlkugZ7"}'
  mnemonic: ""


**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

some words forming mnemonic seed will be placed here you have to write them down and keep them safe
```

Be sure to back up the seed phrase of your validator wallet. It's also recommended to keep an offline copy along with your key files. Remember, your key files cannot be restored and _**must**_ be backed up. See the installation page for instructions.

You should also backup your keyring files.

Change `WALLET_NAME` to the name of your wallet.

```
mkdir ~/keyring_backup
cp ~/.canine/WALLET_NAME.info ~/keyring_backup
cp ~/.canine/keyhash ~/keyring_backup
```

### Setting Up[​](https://docs.jackalprotocol.com/docs/nodes/validators/joining#setting-up) <a href="#setting-up" id="setting-up"></a>

#### Configure Gas Prices[​](https://docs.jackalprotocol.com/docs/nodes/validators/joining#configure-gas-prices) <a href="#configure-gas-prices" id="configure-gas-prices"></a>

As a validator, you'll need to set a minimum gas price like so:

```
GAS="0.02ujkl"
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"$GAS\"/" $HOME/.canine/config/app.toml
```

#### Create Your Validator[​](https://docs.jackalprotocol.com/docs/nodes/validators/joining#create-your-validator) <a href="#create-your-validator" id="create-your-validator"></a>

Before continuing, please note that `commission-max-change` and `commission-max-rate` cannot be changed once you set them. Your `commission-rate` may be changed once per day.

There are a few things you will need to alter in this command. `amount` needs to be changed to what you are starting your self bond as. `from` needs to be the name of your wallet you created earlier. The `moniker`, `details`, `identity`, `website`, and `security-contact` should all be filled with the appropriate information.

```
canined tx staking create-validator \
    --amount 1000000ujkl \
    --commission-max-change-rate 0.10 \
    --commission-max-rate 0.2 \
    --commission-rate 0.1 \
    --from WALLET_NAME \
    --min-self-delegation 1 \
    --moniker "YOUR_MONIKER" \
    --details="YOUR DETAILS" \
    --identity "PGP IDENTITY" \
    --website="https://example.com" \
    --security-contact="your-email@email.com" \
    --pubkey $(canined tendermint show-validator) \
    --chain-id jackal-1 \
    --gas-prices 0.02ujkl
```