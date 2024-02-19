<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 1779.309 | 0.705658 | 1255.584101701 |

<!-- END_TABLE -->

































[🔥OUR VALIDATOR🔥](https://restake.app/shentu/shentuvaloper1u05xu47w2hemqrpejn3dsxg7nqywx068q34y9z)
=

# Shentu Mainnet guide
![shentu](https://github.com/obajay/nodes-Guides/assets/44331529/c0ac1938-4fc2-4275-b11c-5b2dc4cd1710)

[WebSite](https://www.shentu.technology/)\
[GitHub](https://github.com/shentufoundation)
=
[EXPLORER](https://explorer.stavr.tech/Shentu-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O shentum https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Shentu/shentum && chmod +x shentum && ./shentum
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

# Build 17.11.23
```python
cd $HOME
git clone https://github.com/shentufoundation/shentu
cd shentu
git checkout v2.9.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`shentud version --long | head`
- version: 2.9.0
- commit: 3a7e28ae8dec6bbf4d7b35252a745b82a463f781

```python
shentud init STAVRguide --chain-id shentu-2.2
shentud config chain-id shentu-2.2
```    

## Create/recover wallet
```python
shentud keys add <walletname>
            OR
shentud keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.shentud/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Shentu/genesis.json"

```
`sha256sum $HOME/.shentud/config/genesis.json`
+ f42e1d49ca30f69ace60f5eb61416e9393d318083849e83d1fc33df4085462c0

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025uctk\"/;" ~/.shentud/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.shentud/config/config.toml
peers="31a3436fa45d0c0b6df1275783be405d5ca71850@65.108.199.222:26556"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.shentud/config/config.toml
seeds="b3192e1ab0cbb9f439b15c82605379018d96b4f2@3.209.12.186:26656,23419a3d9deedabce1a3cbfa0d1a3e55ef2364a7@34.229.203.57:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.shentud/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.shentud/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.shentud/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.shentud/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.shentud/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.shentud/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.shentud/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.shentud/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.shentud/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Shentu/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/shentud.service > /dev/null <<EOF
[Unit]
Description=shentud
After=network-online.target

[Service]
User=$USER
ExecStart=$(which shentud) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Shentu Mainnet
```python
SNAP_RPC=https://shentu.rpc.m.stavr.tech:443
SEEDS=060027d3bc10ff7ebc1ec315ae5671c541e1568c@shentu.peer.stavr.tech:20016
cp $HOME/.shentud/data/priv_validator_state.json $HOME/.shentud/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.shentud/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.shentud/config/config.toml
shentud tendermint unsafe-reset-all --home $HOME/.shentud --keep-addr-book
mv $HOME/.shentud/priv_validator_state.json.backup $HOME/.shentud/data/priv_validator_state.json
sudo systemctl restart shentud && journalctl -u shentud -f -o cat
```
# SnapShot Mainnet (~3GB) updated every 7 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop shentud
cp $HOME/.shentud/data/priv_validator_state.json $HOME/.shentud/priv_validator_state.json.backup
rm -rf $HOME/.shentud/data
curl -o - -L http://shentu.snapshot.stavr.tech:2/shentud/shentud-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.shentud --strip-components 2
mv $HOME/.shentud/priv_validator_state.json.backup $HOME/.shentud/data/priv_validator_state.json
wget -O $HOME/.shentud/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Shentu/addrbook.json"
sudo systemctl restart shentud && journalctl -u shentud -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable shentud
sudo systemctl restart shentud && sudo journalctl -u shentud -f -o cat
```

### Create validator
```python
shentud tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000uctk \
--pubkey $(shentud tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id shentu-2.2 \
--fees="5000uctk" \
--identity="" \
--website="" \
--details="" -y
```
[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Shentu)
=

## Delete node
```python
sudo systemctl stop shentud
sudo systemctl disable shentud
rm /etc/systemd/system/shentud.service
sudo systemctl daemon-reload
cd $HOME
rm -rf shentu
rm -rf .shentud
rm -rf $(which shentud)
```
#
### Sync Info
```python
shentud status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
shentud status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u shentud -f -o cat
```
### Check Balance
```python
shentud query bank balances shentu...addressjkl1yjgn7z09ua9vms259j
```
