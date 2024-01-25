# Carbon Mainnet guide

![carbon](https://github.com/obajay/nodes-Guides/assets/44331529/b049e2af-464f-4c12-a2fe-335a99df0971)

[WebSite](https://www.carbon.network/)\
[GitHub](https://github.com/Switcheo/carbon-bootstrap)
=
[EXPLORER 1](https://explorer.stavr.tech/Carbon-Mainnet) \
[EXPLORER 2](https://ping.pub/carbon/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
SOOON
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.21.4
```python
ver="1.21.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 22.01.23
```python
cd $HOME && mkdir -p go/bin/
wget https://github.com/Switcheo/carbon-bootstrap/releases/download/v2.37.0/carbond2.37.0-mainnet.linux-amd64.tar.gz
tar -xvzf carbond2.37.0-mainnet.linux-amd64.tar.gz
rm -rf carbond2.37.0-mainnet.linux-amd64.tar.gz
chmod +x carbond
mv carbond /root/go/bin/

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`carbond version --long | head`
- version: 2.37.0
- commit: 14b9e8105b00b94faca280b9f99f104bf2c1d5dd

```python
carbond init STAVR_guide --chain-id carbon-1
carbond config chain-id carbon-1
```    

## Create/recover wallet
```python
carbond keys add <walletname>
  OR
carbond keys add <walletname> --recover
```

## Download Genesis
```python
wget -O genesis.json https://snapshots.polkachu.com/genesis/carbon/genesis.json --inet4-only
mv genesis.json ~/.carbon/config
```
`sha256sum $HOME/.carbon/config/genesis.json`
+ 56943557fcd962364fd35f1f19900146975b9b86d269a3ed2c01a89e5c199705

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.1swth\"/;" ~/.carbon/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.carbon/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.carbon/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.carbon/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:19656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.carbon/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.carbon/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.carbon/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.carbon/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.carbon/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.carbon/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.carbon/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.carbon/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.carbon/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Carbon/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/carbond.service > /dev/null <<EOF
[Unit]
Description=carbond
After=network-online.target

[Service]
User=$USER
ExecStart=$(which carbond) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Carbon Mainnet
```python
SOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable carbond
sudo systemctl restart carbond && sudo journalctl -u carbond -f -o cat
```

### Create validator
```python
carbond tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 100000000swth \
--pubkey $(carbond tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id carbon-1 \
--gas 900000 \
--fees 100000000swth \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Carbon)
=


## Delete node
```python
sudo systemctl stop carbond
sudo systemctl disable carbond
rm /etc/systemd/system/carbond.service
sudo systemctl daemon-reload
cd $HOME
rm -rf carbon-bootstrap
rm -rf .carbon
rm -rf $(which carbond)
```
#
### Sync Info
```python
carbond status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
carbond status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u carbond -f -o cat
```
### Check Balance
```python
carbond query bank balances swth...addressjkl1yjgn7z09ua9vms259j
```
