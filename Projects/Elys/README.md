# Elys Testnet guide

![Logo5](https://user-images.githubusercontent.com/44331529/232235690-19de528b-1c02-4fe3-9cde-3d2864b09f22.png)

[WebSite](https://elys.network) \
[GitHub](https://github.com/elys-network/elys)
=
[EXPLORER](https://explorer.stavr.tech/Elys-Testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O elyss https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Elys/elyss && chmod +x elyss && ./elyss
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
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

# Build 16.03.24
```python
cd $HOME
git clone https://github.com/elys-network/elys elys
cd elys
git checkout v0.29.24
make build
sudo mv $HOME/elys/build/elysd $HOME/go/bin/elysd
```

*******🟢UPDATE🟢******* 16.03.24
```python
cd $HOME/elys && git pull
git checkout v0.29.24
make build
sudo mv $HOME/elys/build/elysd $(which elysd)
elysd version --long | grep -e commit -e version
#commit: a33d9063e43760fae0bc0341f9d1744979f5b02d
#version: v0.29.24
sudo systemctl restart elysd && sudo journalctl -u elysd -f -o cat
```

`elysd version --long`
- version: v0.29.24
- commit: a33d9063e43760fae0bc0341f9d1744979f5b02d

```python
elysd init STAVR_guide --chain-id elystestnet-1
elysd config chain-id elystestnet-1
```    

## Create/recover wallet
```python
elysd keys add <walletname>
  OR
elysd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O genesis.json https://snapshots.polkachu.com/testnet-genesis/elys/genesis.json --inet4-only
mv genesis.json ~/.elys/config

```
`sha256sum $HOME/.elys/config/genesis.json`
+ 4f0ee139d9ce25ead4195bf7afa4e08422d3a409e0e1d3e271e7d8b512bff0a9

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.00025uelys,0.0018ibc/2180E84E20F5679FCC760D8C165B60F42065DEF7F46A72B447CFF1B7DC6C0A65,0.00025ibc/E2D2F6ADCC68AA3384B2F5DFACCA437923D137C14E86FB8A10207CF3BED0C8D4\"|" $HOME/.elys/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.elys/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.elys/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.elys/config/config.toml
seeds=""
CONFIG_TOML="$HOME/.elys/config/config.toml"
sed -i 's/timeout_propose =.*/timeout_propose = "3s"/g' $CONFIG_TOML
sed -i 's/timeout_propose_delta =.*/timeout_propose_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote =.*/timeout_prevote = "1s"/g' $CONFIG_TOML
sed -i 's/timeout_prevote_delta =.*/timeout_prevote_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit =.*/timeout_precommit = "1s"/g' $CONFIG_TOML
sed -i 's/timeout_precommit_delta =.*/timeout_precommit_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/^timeout_commit =.*/timeout_commit = "5s"/g' $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.elys/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.elys/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.elys/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.elys/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.elys/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.elys/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.elys/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.elys/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.elys/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Elys/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/elysd.service > /dev/null <<EOF
[Unit]
Description=elys
After=network-online.target

[Service]
User=$USER
ExecStart=$(which elysd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Elys Testnet
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
sudo systemctl enable elysd
sudo systemctl restart elysd && sudo journalctl -u elysd -f -o cat
```

### Create validator
```python
elysd tx staking create-validator \
  --amount 1000000uelys \
  --commission-max-change-rate "0.2" \
  --commission-max-rate "0.50" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey=$(elysd tendermint show-validator) \
  --moniker 'STAVR_guide' \
  --website "" \
  --identity "" \
  --details "" \
  --chain-id elystestnet-1 \
  --from STAVR1 -y
```

## Delete node
```python
sudo systemctl stop elysd
sudo systemctl disable elysd
rm /etc/systemd/system/elysd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf elys
rm -rf .elys
rm -rf $(which elysd)
```
#
### Sync Info
```python
elysd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
elysd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u elysd -f -o cat
```
### Check Balance
```python
elysd query bank balances elysd...addressjkl1yjgn7z09ua9vms259j
```
