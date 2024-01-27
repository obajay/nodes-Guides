# Desmos Mainnet guide

![desmos](https://github.com/obajay/nodes-Guides/assets/44331529/b0382cfa-6a13-462c-b85a-dcfd6691a8c2)

[WebSite](https://desmos.network/) \
[GitHub](https://github.com/desmos-labs/desmos.git)
=
[EXPLORER 1](https://explorer.stavr.tech/Desmos-Mainnet) \
[EXPLORER 2](https://ping.pub/desmos/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O desmosm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Desmos/desmosm && chmod +x desmosm && ./desmosm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.19.3
```python
ver="1.19.3"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

# Build 20.11.23
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/desmos-labs/desmos.git
cd desmos
git checkout v6.2.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`desmos version --long | grep -e commit -e version`
- version: 6.2.0
- commit: d5cc9d559e3434840160581040d8df2b23b0be7c

```python
desmos init STAVR_guide --chain-id desmos-mainnet
desmos config chain-id desmos-mainnet
```    

## Create/recover wallet
```python
desmos keys add <walletname>
  OR
desmos keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.desmos/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Desmos/genesis.json"
```
`sha256sum $HOME/.desmos/config/genesis.json`
+ 8301452877607c2637c21073066cf2ac6d1fa6b961ffb73ce974dadafeca7b5b

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01udsm\"/;" ~/.desmos/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.desmos/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.desmos/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.desmos/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.desmos/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.desmos/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.desmos/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.desmos/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.desmos/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.desmos/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.desmos/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.desmos/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.desmos/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Desmos/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/desmos.service > /dev/null <<EOF
[Unit]
Description=desmos
After=network-online.target

[Service]
User=$USER
ExecStart=$(which desmos) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Desmos Mainnet
```python
SNAP_RPC=https://desmos.rpc.m.stavr.tech:443
SEEDS=a225d2e2dfe610993d83bf5e25025bde3ef38095@desmos.seed.stavr.tech:1046
cp $HOME/.desmos/data/priv_validator_state.json $HOME/.desmos/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.desmos/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.desmos/config/config.toml
desmos tendermint unsafe-reset-all --home $HOME/.desmos --keep-addr-book
mv $HOME/.desmos/priv_validator_state.json.backup $HOME/.desmos/data/priv_validator_state.json
sudo systemctl restart desmos && journalctl -u desmos -f -o cat
```
# SnapShot Mainnet (~0.9GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop desmos
cp $HOME/.desmos/data/priv_validator_state.json $HOME/.desmos/priv_validator_state.json.backup
rm -rf $HOME/.desmos/data
curl -o - -L https://desmos.snapshot.stavr.tech/desmos/desmos-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.desmos --strip-components 2
curl -o - -L https://desmos.wasm.stavr.tech/wasm-desmos.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.desmos --strip-components 2
mv $HOME/.desmos/priv_validator_state.json.backup $HOME/.desmos/data/priv_validator_state.json
wget -O $HOME/.desmos/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Desmos/addrbook.json"
sudo systemctl restart desmos && journalctl -u desmos -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable desmos
sudo systemctl restart desmos && sudo journalctl -u desmos -f -o cat
```

### Create validator
```python
desmos tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000udsm \
--pubkey $(desmos tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id desmos-mainnet \
--gas 300000 \
--fees 3000udsm \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Desmos)
=


## Delete node
```python
sudo systemctl stop desmos
sudo systemctl disable desmos
rm /etc/systemd/system/desmos.service
sudo systemctl daemon-reload
cd $HOME
rm -rf desmos
rm -rf .desmos
rm -rf $(which desmos)
```
#
### Sync Info
```python
desmos status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
desmos status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u desmos -f -o cat
```
### Check Balance
```python
desmos query bank balances desmos...addressjkl1yjgn7z09ua9vms259j
```
