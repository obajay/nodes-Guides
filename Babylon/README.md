# Babylon Testnet guide


![babllon](https://user-images.githubusercontent.com/44331529/216766002-f883b348-3743-46cf-819c-d4ca72a3d3d8.png)


[WebSite](https://babylonchain.io/) \
[GitHub](https://github.com/babylonchain)
=
[EXPLORER](https://explorer.stavr.tech/humans-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
SOOON
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
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

# Build 03.02.23
```python
cd $HOME
git clone https://github.com/babylonchain/babylon
cd babylon
git checkout v0.5.0
make install
```
`babylond version --long`
- version: v0.5.0
- commit: c8315ce7d302e9997220a92775de29dfaf7f42f7

```python
babylond config chain-id bbn-test2
babylond init STAVRguide --chain-id bbn-test2
```    

## Create/recover wallet
```python
babylond keys add <walletname>
          or 
babylond keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/obajay/nodes-Guides/main/Babylon/genesis.json -O $HOME/.babylond/config/genesis.json

```
`sha256sum $HOME/..babylond/config/genesis.json`
+ sooon

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
SEEDS=""
PEERS="c2db7e074a6ac1fba3c0faf3a06e1cc890473993@rpc.testnet.babylonchain.io:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.babylond/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ubbn\"/;" ~/.babylond/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.babylond/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.babylond/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.babylond/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.babylond/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.babylond/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.babylond/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.babylond/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.babylond/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.babylond/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.babylond/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.babylond/config/addrbook.json "SOOOOOOOOON"
```

# Create a service file
```python
sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
[Unit]
Description=babylond
After=network-online.target

[Service]
User=$USER
ExecStart=$(which babylond) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload && sudo systemctl enable babylond
sudo systemctl restart babylond && sudo journalctl -u babylond -f -o cat
```

### Create validator
```python
babylond tx staking create-validator \
  --amount 1000000ubbn \
  --from wallet \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(babylond tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id bbn-test2 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop babylond && \
sudo systemctl disable babylond && \
rm /etc/systemd/system/babylond.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf babylon && \
rm -rf .babylond && \
rm -rf $(which babylond)
```
#
### Sync Info
```python
source $HOME/.bash_profile
babylond status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
babylond status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u babylond -f -o cat
```
### Check Balance
```python
babylond query bank balances bbn...address1yjgn7z09ua9vms259j
```
