<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 67574.205 | 0.01525306 | 1030.71341442 |

<!-- END_TABLE -->

























[🔥OUR VALIDATOR🔥](https://restake.app/decentr/decentrvaloper1yv2ndyf63nk65gqhhrfep5pselghrwvveaa5d4)
=

# Decentr Mainnet guide
![deventr](https://github.com/obajay/nodes-Guides/assets/44331529/0e5a41fa-99fd-414e-b8a0-a09faa2257fd)


[WebSite](https://decentr.net/)\
[GitHub](https://github.com/Decentr-net/decentr)
=
[EXPLORER](https://explorer.stavr.tech/Decentr-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O decentrm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Decentr/decentrm && chmod +x decentrm && ./decentrm
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

# Build 20.11.23
```python
cd $HOME
git clone https://github.com/Decentr-net/decentr
cd decentr
git checkout v1.6.4
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`decentrd version --long | grep -e commit -e version`
- version: 1.6.4
- commit: 2db736296c1089f3fbdf7328e7309e9fc7be1f13

```python
decentrd init STAVRguide --chain-id mainnet-3
decentrd config chain-id mainnet-3
```    

## Create/recover wallet
```python
decentrd keys add <walletname>
            OR
decentrd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.decentr/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Decentr/genesis.json"

```
`sha256sum $HOME/.decentr/config/genesis.json`
+ effc0b3f925f89dda69d2e8f40fb8ca92b3f92b714cc364b5ec11d4349138853

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0udec\"/;" ~/.decentr/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.decentr/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.decentr/config/config.toml
seeds="5f5cfac5c38506fbb4275c19e87c4107ec48808d@seeds.nodex.one:11310"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.decentr/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.decentr/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.decentr/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.decentr/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.decentr/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.decentr/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.decentr/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.decentr/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.decentr/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Decentr/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/decentrd.service > /dev/null <<EOF
[Unit]
Description=decentrd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which decentrd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Decentr Mainnet
```python
SNAP_RPC=https://decentr.rpc.m.stavr.tech:443
SEEDS=1f5497f2b4f6adb3b803c17c3b005f637fcaec2d@decentr.peer.stavr.tech:1066
cp $HOME/.decentr/data/priv_validator_state.json $HOME/.decentr/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.decentr/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.decentr/config/config.toml
decentrd tendermint unsafe-reset-all --home $HOME/.decentr --keep-addr-book
mv $HOME/.decentr/priv_validator_state.json.backup $HOME/.decentr/data/priv_validator_state.json
sudo systemctl restart decentrd && journalctl -u decentrd -f -o cat
```
# SnapShot Mainnet (~0.3 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop decentrd
cp $HOME/.decentr/data/priv_validator_state.json $HOME/.decentr/priv_validator_state.json.backup
rm -rf $HOME/.decentr/data
curl -o - -L http://decentr.snapshot.stavr.tech:9/decentr/decentr-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.decentr --strip-components 2
mv $HOME/.decentr/priv_validator_state.json.backup $HOME/.decentr/data/priv_validator_state.json
wget -O $HOME/.decentr/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Decentr/addrbook.json"
sudo systemctl restart decentrd && journalctl -u decentrd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable decentrd
sudo systemctl restart decentrd && sudo journalctl -fu decentrd -o cat
```

### Create validator
```python
decentrd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000udec \
--pubkey $(decentrd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id mainnet-3 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Decentr)
=

## Delete node
```python
sudo systemctl stop decentrd
sudo systemctl disable decentrd
rm /etc/systemd/system/decentrd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf decentr
rm -rf .decentr
rm -rf $(which decentrd)
```
#
### Sync Info
```python
decentrd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
decentrd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u decentrd -f -o cat
```
### Check Balance
```python
decentrd query bank balances decentr...addressjkl1yjgn7z09ua9vms259j
```

