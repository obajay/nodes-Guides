### Download the nodesource PPA for Node.js
```
curl https://deb.nodesource.com/setup_16.x | sudo bash
```

### Install the Yarn package manager
```bash
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

# Update Ubuntu
sudo apt-get update
sudo apt upgrade -y

# Install Node.js, Yarn, and build tools
# Install jq for formatting of JSON data
# We'll need git below.
sudo apt install nodejs=16.* yarn build-essential git jq -y

# verify installation
node --version | grep 16
yarn --version
```
## Install Go

`Agoric's Cosmos integration is built using Go and requires Go version 1.17+. In this example, we will be installing Go on the above Ubuntu 20.04 with Node.js installed:`

```bash
# First remove any existing old Go installation
sudo rm -rf /usr/local/go

# Download and verify go
curl -L -o /tmp/go1.18.7.linux-amd64.tar.gz https://go.dev/dl/go1.18.7.linux-amd64.tar.gz
sha256sum --check <<EOF
6c967efc22152ce3124fc35cdf50fc686870120c5fd2107234d05d450a6105d8  /tmp/go1.18.7.linux-amd64.tar.gz
EOF

# install
sudo tar -C /usr/local -xzf /tmp/go1.18.7.linux-amd64.tar.gz

# Update environment variables to include go
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile

# Verify that Go is installed:
go version | grep 1.18
```

## Install Agoric SDK
```bash
cd # Writeable directory of your choice. $HOME will do.
git clone https://github.com/Agoric/agoric-sdk -b pismoA
cd agoric-sdk

# Install and build Agoric Javascript packages
yarn install && yarn build

# Install and build Agoric Cosmos SDK support
(cd packages/cosmic-swingset && make)
```

`agd version --long`
The output should start with:
```
name: agoriccosmos
server_name: ag-cosmos-helper
version: 0.32.2
commit: 2c812d221
build_tags: ',ledger'
go: go version go1.18.1 linux/amd64
```

## Configure agd.service
To use systemd, we will create a service file:
```bash
sudo tee <<EOF >/dev/null /etc/systemd/system/agd.service
[Unit]
Description=Agoric Cosmos daemon
After=network-online.target

[Service]
# OPTIONAL: turn on JS debugging information.
#SLOGFILE=.agoric/data/chain.slog
User=$USER
# OPTIONAL: turn on Cosmos nondeterminism debugging information
#ExecStart=$(which agd) start --log_level=info --trace-store=.agoric/data/kvstore.trace
ExecStart=$(which agd) start --log_level=info
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable agd
sudo systemctl daemon-reload
```

`systemctl status agd`
![1](https://user-images.githubusercontent.com/44331529/198227093-831a8411-ac56-449f-85b4-cd8047fb3b27.png)

```
ag0 tendermint show-validator
agd tendermint show-validator
```

### Disable ag0 once it stops
`ag0` should stop on its own at the block height in the software upgrade governance proposal.

Once that happens, disable it:

`sudo systemctl disable ag0` \
Start agd to complete the upgrade \
`sudo systemctl start agd`


[Original link](https://github.com/Agoric/agoric-sdk/wiki/ag0-to-agd-upgrade-for-mainnet-1-launch)
