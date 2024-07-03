# Creating Storage Provider

## Pre-Requisites[​](https://docs.jackalprotocol.com/docs/nodes/install#pre-requisites) <a href="#pre-requisites" id="pre-requisites"></a>

There are a few things needed before installing.

While logged in as the 'root' or 'admin' user, we add a 'jackal' user and give them root privileges.

#### Enable Firewall Rules[​](https://docs.jackalprotocol.com/docs/nodes/install#enable-firewall-rules) <a href="#enable-firewall-rules" id="enable-firewall-rules"></a>

Enabling the firewall is important to ensure your hardware remains secure. The following commands will add rules required for access on both validators and providers:

```
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
```

Additional ports are required if you are running a validator:

```
sudo ufw allow 26657
sudo ufw allow 26658
```

The only additional port required for a provider is 3333:

```
sudo ufw allow 3333
```

If you are running a combined validator/provider, you need to allow all of the above ports. After adding ports to the rules list, you will need to start the firewall:

```
sudo ufw enable
```

After starting the firewall, verify all of the required rules are in place by running:

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
1317                       ALLOW       Anywhere
3333                       ALLOW       Anywhere
26657                      ALLOW       Anywhere
26658                      ALLOW       Anywhere
```

#### Create Jackal user[​](https://docs.jackalprotocol.com/docs/nodes/install#create-jackal-user) <a href="#create-jackal-user" id="create-jackal-user"></a>

```
sudo adduser --gecos "" jackal
sudo usermod -aG sudo jackal
```

Log in as the jackal user to complete the below steps:

```
sudo su - jackal
```

#### Installing required tools[​](https://docs.jackalprotocol.com/docs/nodes/install#installing-required-tools) <a href="#installing-required-tools" id="installing-required-tools"></a>

This will install the necessary tools to build the jackal chain source, along with lz4 compression tool and jquery tool.

```
sudo apt update
sudo apt install build-essential lz4 jq
```

#### Installing Go[​](https://docs.jackalprotocol.com/docs/nodes/install#installing-go) <a href="#installing-go" id="installing-go"></a>

Follow more in-depth instructions to install Go v1.22 or higher [here](https://golang.org/doc/install).

On Ububtu you can install it with:

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

Restarting the shell with youre profile settings or just rebasing them like so is required.

```
source ~/.profile
```

## Installing[​](https://docs.jackalprotocol.com/docs/nodes/providers/setting\_up#installing) <a href="#installing" id="installing"></a>

{% hint style="success" %}
Check latest version [here](https://github.com/JackalLabs/canine-provider/releases).
{% endhint %}

{% hint style="info" %}
'Setting Up' instructions must be followed fully to add necessary golang path info to the current users \~/.profile. If these steps are skipped, 'make install' will not build jprovd--the provider daemon. Please ensure to perform the below steps as the 'jackal' user you previously made.
{% endhint %}

Install make and confirm installation.

```
sudo apt update

sudo apt install make

make --version
```

Build jprovd and source the .profile to ensure your shell can find jprovd. Confirm installation.

```
git clone https://github.com/JackalLabs/sequoia.git

cd sequoia

git pull

git checkout {version}

make install

source ~/.profile

sequoia version
```

### Initializing[​](https://docs.jackalprotocol.com/docs/nodes/providers/setting\_up#initializing) <a href="#initializing" id="initializing"></a>

