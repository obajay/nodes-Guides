<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 1578105.6 | 0.00264055 | 4167.06677815 |

<!-- END_TABLE -->







































































































































[🔥OUR VALIDATOR🔥](https://restake.app/gravitybridge/gravityvaloper1qz50nzevfjqaftt67twfr2tzajc27uv7n5ttfv)
=


# GravityBridge Mainnet guide
![grav](https://github.com/obajay/nodes-Guides/assets/44331529/1a4043f1-c011-49e2-a778-a30bbd543635)

[WebSite](https://www.gravitybridge.net/)\
[GitHub](https://github.com/Gravity-Bridge)
=
[EXPLORER 1](https://explorer.stavr.tech/GravityBridge/staking) \
[EXPLORER 2](https://dev.mintscan.io/gravity-bridge/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  16| 32GB | 250GB    |


# 1) Auto_install script
```python
wget -O gravitym https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Gravitybridge/gravitym && chmod +x gravitym && ./gravitym
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.19
```python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 15.11.23
```python
cd $HOME
mkdir gravity-bin && cd gravity-bin
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.11.1/gravity-linux-amd64
mv gravity-linux-amd64 gravity
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.11.1/gbt
chmod +x *
sudo mv * /usr/local/bin/

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`gravity version --long | head`
- version: v1.11.1
- commit: d8f348d6035de64b23092bcc6a9b04403a125451

```python
gravity init STAVRguide --chain-id gravity-bridge-3
gravity config chain-id gravity-bridge-3
gbt init


```    

## Create/recover wallet
```python
gravity keys add <wallet_name> --algo secp256k1 --coin-type 118
gravity keys add <wallet_orchestrator_name>
gravity eth_keys add <wallet_eth_name>
```

##Key set for orchestrator
```python
gbt keys set-ethereum-key --key <your Ethereum PRIVATE key>
gbt keys set-orchestrator-key --phrase "<your mnemonic from orchestrator wallet>"
```


## Download Genesis
```python
wget -O $HOME/.gravity/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Gravitybridge/genesis.json"

```
`sha256sum $HOME/.gravity/config/genesis.json`
+ 66401e3c2d3f2679f82bec6a53c9b9bc38737c1ecb700c0a85f0188f8840ddeb

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001ugraviton\"/;" ~/.gravity/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.gravity/config/config.toml
peers="c189b7217b037e50b3456440963f91d027a4df5a@65.108.199.222:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.gravity/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.gravity/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.gravity/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.gravity/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gravity/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gravity/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.gravity/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.gravity/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.gravity/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.gravity/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Gravitybridge/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/gravity.service > /dev/null <<EOF
[Unit]
Description=gravity
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gravity) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Gravity Mainnet
```python
SOON
```
# SnapShot Mainnet (~0.2GB) updated every 5 hours  
```python
SOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable gravity
sudo systemctl restart gravity && sudo journalctl -u gravity -f -o cat
```

###Create the service file Orchestrator
```python
sudo tee /etc/systemd/system/orchestrator.service > /dev/null <<EOF
[Unit]
Description=Gravity Bridge Orchestrator
Requires=network.target
[Service]
Type=simple
TimeoutStartSec=10s
Restart=on-failure
RestartSec=10
ExecStart=$(which gbt) orchestrator \
--fees 5000ugraviton \
--gravity-contract-address 0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906 \
--ethereum-rpc "https://eth.althea.net/"
[Install]
WantedBy=default.target
EOF
```
###Load service and start
```python
sudo systemctl daemon-reload && sudo systemctl enable orchestrator
sudo systemctl restart orchestrator && journalctl -fu orchestrator -o cat
```

### Create validator
```python
gravity tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.2 \
--min-self-delegation "1" \
--amount 1000000000000000000ugraviton \
--pubkey $(gravity tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id gravity-bridge-3 \
--fees="500ugraviton" \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop gravity
sudo systemctl disable gravity
rm /etc/systemd/system/gravity.service
sudo systemctl daemon-reload
cd $HOME
rm -rf gravity-bin
rm -rf .gravity
rm -rf $(which gravity)
```
#
### Sync Info
```python
gravity status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
gravity status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u gravity -f -o cat
```
### Check Balance
```python
gravity query bank balances gravity...addressjkl1yjgn7z09ua9vms259j
```
