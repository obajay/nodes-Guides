<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 82333.9 | 0.04571872 | 3764.203124106 |

<!-- END_TABLE -->




































































[🔥OUR VALIDATOR🔥](https://restake.app/regen/regenvaloper1adzpsnvegat0mdcd2yn3hpy8hly9t20cwzjmj6)
=

# Regen Network Mainnet guide

![regen](https://github.com/obajay/nodes-Guides/assets/44331529/8a3f6180-81b5-46ff-aedf-62546514087a)

[WebSite](https://www.regen.network/) \
[GitHub](https://github.com/regen-network/)
=
[EXPLORER 1](https://explorer.stavr.tech/Regen-Mainnet/) \
[EXPLORER 2](https://ping.pub/regen)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O regenm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Regen/regenm && chmod +x regenm && ./regenm
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

# Build 19.12.23
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/regen-network/regen-ledger/
cd regen-ledger
git checkout v5.1.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`regen version --long | grep -e commit -e version`
- version: 5.1.0
- commit: c08da230dae102c3051b383e17532e20b238fb71

```python
regen init STAVR_guide --chain-id regen-1
regen config chain-id regen-1
```    

## Create/recover wallet
```python
regen keys add <walletname>
  OR
regen keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.regen/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Regen/genesis.json"
```
`sha256sum $HOME/.regen/config/genesis.json`
+ 98d2c9dd90586078099ca750328ac963ec4f1e2b9e95de05298e37bf9cb0f78d

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.1uregen\"/;" ~/.regen/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.regen/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.regen/config/config.toml
peers="d35d652b6cb3bf7d6cb8d4bd7c036ea03e7be2ab@116.203.182.185:26656,ebc272824924ea1a27ea3183dd0b9ba713494f83@regen-mainnet-peer.autostake.com:27216"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.regen/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.regen/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.regen/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.regen/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.regen/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.regen/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.regen/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.regen/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.regen/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.regen/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Regen/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/regen.service > /dev/null <<EOF
[Unit]
Description=regen
After=network-online.target

[Service]
User=$USER
ExecStart=$(which regen) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Regen Network Mainnet
```python
SNAP_RPC=https://regen.rpc.m.stavr.tech:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.regen/config/config.toml
regen tendermint unsafe-reset-all
wget -O $HOME/.regen/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Regen/addrbook.json"
sudo systemctl restart regen && sudo journalctl -u regen -f -o cat
```
# SnapShot Mainnet (~0.5GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop regen
cp $HOME/.regen/data/priv_validator_state.json $HOME/.regen/priv_validator_state.json.backup
rm -rf $HOME/.regen/data
curl -o - -L https://regen.snapshot.stavr.tech/regen/regen-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.regen --strip-components 2
mv $HOME/.regen/priv_validator_state.json.backup $HOME/.regen/data/priv_validator_state.json
wget -O $HOME/.regen/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Regen/addrbook.json"
sudo systemctl restart regen && journalctl -u regen -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable regen
sudo systemctl restart regen && sudo journalctl -u regen -f -o cat
```

### Create validator
```python
regen tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000uregen \
--pubkey $(regen tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id regen-1 \
--gas 300000 \
--fees 30000umed \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Regen)
=


## Delete node
```python
sudo systemctl stop regen
sudo systemctl disable regen
rm /etc/systemd/system/regen.service
sudo systemctl daemon-reload
cd $HOME
rm -rf regen-ledger
rm -rf .regen
rm -rf $(which regen)
```
#
### Sync Info
```python
regen status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
regen status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u regen -f -o cat
```
### Check Balance
```python
regen query bank balances regen...addressjkl1yjgn7z09ua9vms259j
```
