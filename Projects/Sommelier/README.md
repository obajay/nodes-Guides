<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 13492.7 | 0.224582 | 3030.227813674 |

<!-- END_TABLE -->

































































































[🔥OUR VALIDATOR🔥](https://restake.app/sommelier/sommvaloper1fes322u0x7422ugc05xkxeav2zzd3kfgf898f4)
=

# Sommelier Mainnet guide

![somm](https://github.com/obajay/nodes-Guides/assets/44331529/f060644e-ca9f-4772-ae00-0afa987c4bf8)

[WebSite](https://www.sommelier.finance/) \
[GitHub](https://github.com/PeggyJV/sommelier)
=
[EXPLORER 1](https://explorer.stavr.tech/Sommelier-Mainnet/) \
[EXPLORER 2](https://dev.mintscan.io/sommelier/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O sommm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sommelier/sommm && chmod +x sommm && ./sommm
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

# Build 09.02.24
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/PeggyJV/sommelier
cd sommelier
git checkout v7.0.1
make install

```
*******🟢UPDATE🟢******* 09.02.24
```python
cd $HOME/sommelier
git fetch --all
git checkout v7.0.1
make install
sommelier version --long | grep -e commit -e version
#commit: 25bf250b9f9b2819f81f78bd856f60871d5139bd
#version: v7.0.1
sudo systemctl restart sommelier && sudo journalctl -u sommelier -f -o cat
```

`sommelier version --long | grep -e commit -e version`
- version: v7.0.1
- commit: 25bf250b9f9b2819f81f78bd856f60871d5139bd

```python
sommelier init STAVR_guide --chain-id sommelier-3
sommelier config chain-id sommelier-3
```    

## Create/recover wallet
```python
sommelier keys add <walletname>
  OR
sommelier keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.sommelier/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sommelier/genesis.json"
```
`sha256sum $HOME/.sommelier/config/genesis.json`
+ 68e918e6ee379ec86f6907e8a75ce7958f2c7e7a6addc635dd048f65b6c6c8a8

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usomm\"/;" $HOME/.sommelier/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sommelier/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sommelier/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sommelier/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sommelier/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.sommelier/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.sommelier/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sommelier/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sommelier/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sommelier/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sommelier/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sommelier/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.sommelier/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sommelier/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/sommelier.service > /dev/null <<EOF
[Unit]
Description=sommelier
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sommelier) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Sommelier Mainnet
```python
SNAP_RPC=https://somm.rpc.m.stavr.tech:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.sommelier/config/config.toml
sommelier tendermint unsafe-reset-all
wget -O $HOME/.sommelier/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sommelier/addrbook.json"
sudo systemctl restart sommelier && sudo journalctl -u sommelier -f -o cat
```
# SnapShot Mainnet (~0.4GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop sommelier
cp $HOME/.sommelier/data/priv_validator_state.json $HOME/.sommelier/priv_validator_state.json.backup
rm -rf $HOME/.sommelier/data
curl -o - -L https://sommelier.snapshot.stavr.tech/sommelier/sommelier-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.sommelier --strip-components 2
mv $HOME/.sommelier/priv_validator_state.json.backup $HOME/.sommelier/data/priv_validator_state.json
wget -O $HOME/.sommelier/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sommelier/addrbook.json"
sudo systemctl restart sommelier && journalctl -u sommelier -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable sommelier
sudo systemctl restart sommelier && sudo journalctl -u sommelier -f -o cat
```

### Create validator
```python
sommelier tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000usomm \
--pubkey $(sommelier tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id sommelier-3 \
--gas 300000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Sommelier)
=


## Delete node
```python
sudo systemctl stop sommelier
sudo systemctl disable sommelier
rm /etc/systemd/system/sommelier.service
sudo systemctl daemon-reload
cd $HOME
rm -rf sommelier
rm -rf .sommelier
rm -rf $(which sommelier)
```
#
### Sync Info
```python
sommelier status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
sommelier status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u sommelier -f -o cat
```
### Check Balance
```python
sommelier query bank balances somm...addressjkl1yjgn7z09ua9vms259j
```
