# Creating Storage Provider

## Pre-Requisites

There are a few things needed before installing.

While logged in as the `root` or `admin` user, we add a `jackal` user and give them root privileges.

#### Enable Firewall Rules

Enabling the firewall is important to ensure your hardware remains secure. The following commands will add rules required for access on both validators and providers:

```
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
```

The only additional ports required for a provider:

```
sudo ufw allow 3333
sudo ufw allow 4005
sudo ufw allow 4001
```

If you are running a combined validator/provider, you need to allow all of the above ports. After adding ports to the rules list, you will need to start the firewall:

```
sudo ufw enable
```

After starting the firewall, verify all the required rules are in place by running:

```
sudo ufw status verbose
```

Your output should be similar to the following:

```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80                         ALLOW       Anywhere
443                        ALLOW       Anywhere
3333                       ALLOW       Anywhere
4005                       ALLOW       Anywhere
4001                       ALLOW       Anywhere
```

#### Create Jackal user

```
sudo adduser --gecos "" jackal
sudo usermod -aG sudo jackal
```

Log in as the `jackal` user to complete the below steps:

```
sudo su - jackal
```

#### Installing required tools

This will install the necessary tools to build the jackal chain source, along with `lz4` compression tool and `jquery` tool.

```
sudo apt update
sudo apt install build-essential lz4 jq
```

#### Installing Go

Follow more in-depth instructions to install Go v1.22 or higher [here](https://golang.org/doc/install).

On Ubuntu you can install it with:

```
GOVER=$(curl https://go.dev/VERSION?m=text)
wget https://golang.org/dl/${GOVER}.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf ${GOVER}.linux-amd64.tar.gz
```

Add the following golang path info to the current users `~/.profile`.

Also add it to the skeleton profile so all new users have it. `/etc/skel/.profile`

```
# add environmental variables for Go
if [ -f "/usr/local/go/bin/go" ] ; then
    export GOROOT=/usr/local/go
    export GOPATH=${HOME}/go
    export GOBIN=$GOPATH/bin
    export PATH=${PATH}:${GOROOT}/bin:${GOBIN}
    export GO111MODULE=on
fi
```

Restarting the shell with your profile settings or just rebasing them like so is required.

```
source ~/.profile
```

## Installing

{% hint style="success" %}
Check latest version [here](https://github.com/JackalLabs/canine-provider/releases).
{% endhint %}

{% hint style="info" %}
`Setting Up` instructions must be followed fully to add necessary golang path info to the current users `~/.profile`. If these steps are skipped, `make install` will not build `sequoia` (the provider daemon). Please ensure to perform the below steps as the `jackal` user you previously made.
{% endhint %}

Install make and confirm installation.

```
sudo apt update

sudo apt install make

make --version
```

Build `sequoia` and source the .profile to ensure your shell can find `sequoia`. Confirm installation.

```
git clone https://github.com/JackalLabs/sequoia.git

cd sequoia

git pull

git checkout {version}

make install

source ~/.profile

sequoia version
```

### Initializing

```shell
sequoia init
```

This will create a `~/.sequoia` directory. In this directory you can find your `config.yaml` and your new wallet keys.

This config file should look something like this:
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

To learn what each field does please head over to [Sequoia Config](providing-storage/sequoia-config.md)

The main things to pay attention to are your `domain` and your `rpc_addr`/`grpc_addr`. Without these, set correctly your provider will simply not work.

After you have updated the config, you can run `sequoia wallet address` to check the address of the newly created provider, send it the required amount of tokens for collateral (10k JKL) and some dust for gas. 

### Starting Up
```shell
sequoia start
```

Will set up your provider on chain and start running. It is also suggested to create a service to run this in the background.

You can check your provider details on chain by going to `https://api.jackalprotocol.com/jackal/canine-chain/storage/providers/{provider_address}`