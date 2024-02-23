<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 500359.6 | 0.072285 | 36168.495805435299449851 |

<!-- END_TABLE -->





[🔥OUR VALIDATOR🔥](https://restake.app/planq/plqvaloper1htduzp9w4u3nmjjdtwmctvqsx9z74xecc34vnl)
=

# PlanQ Mainnet guide
![planq](https://github.com/obajay/nodes-Guides/assets/44331529/6af6fccb-5aa5-4b44-bb20-6f2e722fd2f7)


[WebSite](https://planq.network/)\
[GitHub](https://github.com/planq-network/planq.git)
=
[EXPLORER](https://explorer.stavr.tech/Planq-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O planqm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Planq/planqm && chmod +x planqm && ./planqm
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

# Build 06.02.24
```python
cd $HOME
git clone https://github.com/planq-network/planq.git
cd planq
git checkout v1.1.0
make install

```
*******🟢UPDATE🟢******* 06.02.24
```python
cd $HOME/planq
git pull
git checkout v1.1.0
make install
planqd version --long | grep -e commit -e version
#version: 1.1.0
#commit: 226bd3624e139fb71ba5f4fd7c523e5965f4c176
sudo systemctl restart planqd && sudo journalctl -u planqd -f -o cat
```

`planqd version --long | grep -e commit -e version`
- version: 1.1.0
- commit: 226bd3624e139fb71ba5f4fd7c523e5965f4c176

```python
planqd init STAVR_guide --chain-id planq_7070-2
planqd config chain-id planq_7070-2
```    

## Create/recover wallet
```python
planqd keys add <walletname>
            OR
planqd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.planqd/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Planq/genesis.json"

```
`sha256sum $HOME/.planqd/config/genesis.json`
+ a4bca4e9d4de3ee747452aa5dcd80acebb6a69e99dd19b5ce0af1c6606d847f7

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0aplanq\"/;" ~/.planqd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.planqd/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.planqd/config/config.toml
seeds="5f5cfac5c38506fbb4275c19e87c4107ec48808d@seeds.nodex.one:10710"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.planqd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.planqd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.planqd/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.planqd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.planqd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.planqd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.planqd/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.planqd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.planqd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Planq/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/planqd.service > /dev/null <<EOF
[Unit]
Description=planqd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which planqd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Planq Mainnet
```python
SNAP_RPC=https://planq.rpc.m.stavr.tech:443
SEEDS=192ff55d15d7ad9fc9ded5c5a9f4393beba9b222@planq.peer.stavr.tech:1076
cp $HOME/.planqd/data/priv_validator_state.json $HOME/.planqd/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.planqd/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.planqd/config/config.toml
planqd tendermint unsafe-reset-all --home $HOME/.planqd --keep-addr-book
mv $HOME/.planqd/priv_validator_state.json.backup $HOME/.planqd/data/priv_validator_state.json
sudo systemctl restart planqd && journalctl -u planqd -f -o cat
```
# SnapShot Mainnet (~0.3 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop planqd
cp $HOME/.planqd/data/priv_validator_state.json $HOME/.planqd/priv_validator_state.json.backup
rm -rf $HOME/.planqd/data
curl -o - -L http://planq.snapshot.stavr.tech:8/planqd/planqd-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.planqd --strip-components 2
mv $HOME/.planqd/priv_validator_state.json.backup $HOME/.planqd/data/priv_validator_state.json
wget -O $HOME/.planqd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Planq/addrbook.json"
sudo systemctl restart planqd && journalctl -u planqd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable planqd
sudo systemctl restart planqd && sudo journalctl -u planqd -f -o cat
```

### Create validator
```python
planqd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000000000000000aplanq \
--pubkey $(planqd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id planq_7070-2 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Planq)
=

## Delete node
```python
sudo systemctl stop planqd
sudo systemctl disable planqd
rm /etc/systemd/system/planqd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf planq
rm -rf .planqd
rm -rf $(which planqd)
```
#
### Sync Info
```python
planqd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
planqd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u planqd -f -o cat
```
### Check Balance
```python
planqd query bank balances plq...addressjkl1yjgn7z09ua9vms259j
```
