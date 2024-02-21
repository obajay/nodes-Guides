<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 650823.1 | 0.10904 | 70965.750824000 |

<!-- END_TABLE -->

# ODIN Mainnet guide

![odin](https://github.com/obajay/nodes-Guides/assets/44331529/9bd2cc77-167e-49c2-aec2-41e8c18a710f)


[WebSite](https://odinprotocol.io/) \
[GitHub](https://github.com/ODIN-PROTOCOL/odin-core)
=
[EXPLORER 1](https://explorer.stavr.tech/Odin-Mainnet/) \
[EXPLORER 2](https://mainnet.odinprotocol.io/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O odinm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Odin/odinm && chmod +x odinm && ./odinm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.20.5
```python
ver="1.20.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 13.10.22
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/ODIN-PROTOCOL/odin-core.git
cd odin-core
git fetch --tags
git checkout v0.7.9
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`odind version --long | grep -e commit -e version`
- version: 0.7.9
- commit: 6d52fecd7b91c1f848f44efbe1fe648fb2b8ae80

```python
odind init STAVR_guide --chain-id odin-mainnet-freya
odind config chain-id odin-mainnet-freya
```    

## Create/recover wallet
```python
odind keys add <walletname>
  OR
odind keys add <walletname> --recover
```

## Download Genesis
```python
wget -O genesis.json https://snapshots.polkachu.com/genesis/odin/genesis.json --inet4-only
mv genesis.json ~/.odin/config
```
`sha256sum $HOME/.odin/config/genesis.json`
+ ee3e61e92c717eef5ebe3062ae4f314ff5e69c186210174d024998cbc561e222

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0loki\"/;" ~/.odin/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.odin/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.odin/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.odin/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:16856"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.odin/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.odin/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.odin/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.odin/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.odin/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.odin/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.odin/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.odin/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.odin/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Odin/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/odind.service > /dev/null <<EOF
[Unit]
Description=odind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which odind) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync ODIN Mainnet
```python
SOOON
```
# SnapShot Mainnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable odind
sudo systemctl restart odind && sudo journalctl -u odind -f -o cat
```

### Create validator
```python
odind tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.1 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000loki \
--pubkey $(odind tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id odin-mainnet-freya \
--gas 300000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Odin)
=


## Delete node
```python
sudo systemctl stop odind
sudo systemctl disable odind
rm /etc/systemd/system/odind.service
sudo systemctl daemon-reload
cd $HOME
rm -rf odin-core
rm -rf .odin
rm -rf $(which odind)
```
#
### Sync Info
```python
odind status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
odind status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u odind -f -o cat
```
### Check Balance
```python
odind query bank balances odin...addressjkl1yjgn7z09ua9vms259j
```
