# Cascadia Testnet guide

![cascadia](https://user-images.githubusercontent.com/44331529/232544057-cd6fdd6c-1fc4-4048-87db-030364cec75a.png)

[WebSite](https://www.cascadia.foundation/)\
[GitHub](https://github.com/cascadiafoundation/cascadia)
=
[EXPLORER](https://explorer.stavr.tech/Cascadia-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O cascadm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cascadia/cascadm && chmod +x cascadm && ./cascadm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.20.3
```python
ver="1.20.3"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 25.01.24
```python
cd $HOME
git clone https://github.com/CascadiaFoundation/cascadia
cd cascadia
git checkout v0.3.0
make install

```
*******🟢UPDATE🟢******* 25.01.24
```python
cd $HOME/cascadia
git pull
git checkout v0.3.0
cascadiad version --long | grep -e version -e commit
#commit: fcc9d9e2c97965602043ef98a1405c2a3b1888a9
#version: 0.3.0
sudo systemctl restart cascadiad && sudo journalctl -u cascadiad -f -o cat
```

`cascadiad version --long | grep -e version -e commit`
- version: 0.3.0
- commit: fcc9d9e2c97965602043ef98a1405c2a3b1888a9

```python
cascadiad init STAVR_guide --chain-id cascadia_11029-1
cascadiad config chain-id cascadia_11029-1
```    

## Create/recover wallet
```python
cascadiad keys add <walletname>
  OR
cascadiad keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.cascadiad/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cascadia/genesis.json"

```
`sha256sum $HOME/.cascadiad/config/genesis.json`
+ 9e7a698f532042cb966c4cda3ed6eb0aab5ce4882ebfb8d334b27dd84fa497bd

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025aCC\"/;" ~/.cascadiad/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.cascadiad/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.cascadiad/config/config.toml
peers="d1ed80e232fc2f3742637daacab454e345bbe475@54.204.246.120:26656,0c96a6c328eb58d1467afff4130ab446c294108c@34.239.67.55:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.cascadiad/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.cascadiad/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.cascadiad/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.cascadiad/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cascadiad/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.cascadiad/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.cascadiad/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cascadia/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/cascadiad.service > /dev/null <<EOF
[Unit]
Description=Cascadia Foundation
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cascadiad) start --chain-id=cascadia_11029-1
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Cascadian Testnet
```python
SOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable cascadiad
sudo systemctl restart cascadiad && sudo journalctl -u cascadiad -f -o cat
```

### Create validator
```python
cascadiad tx staking create-validator \
  --amount "10000000000000000000"aCC \
  --from wallet \
  --commission-max-change-rate "0.2" \
  --commission-max-rate "0.5" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(cascadiad tendermint show-validator) \
  --moniker "STAVR_guide" \
  --chain-id cascadia_11029-1 \
  --details="" \
  --identity="" \
  --gas-prices 7aCC \
  --gas 250000 \
  --website="" -y
```

## Delete node
```python
sudo systemctl stop cascadiad
sudo systemctl disable cascadiad
rm /etc/systemd/system/cascadiad.service
sudo systemctl daemon-reload
cd $HOME
rm -rf cascadia
rm -rf .cascadiad
rm -rf $(which cascadiad)
```
#
### Sync Info
```python
cascadiad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
cascadiad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u cascadiad -f -o cat
```
### Check Balance
```python
cascadiad query bank balances cascadiad...addressjkl1yjgn7z09ua9vms259j
```
