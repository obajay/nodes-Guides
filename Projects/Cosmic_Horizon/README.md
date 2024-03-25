[🔥OUR VALIDATOR🔥](https://restake.app/qwoyn/qwoynvaloper1d8mhdkf2tzdszt59hvl2uq65h9l6r68y0usvc0)
=

# Qwoyn Mainnet guide
![qwoyn](https://github.com/obajay/nodes-Guides/assets/44331529/5c7c0782-b480-4873-9648-1e4e6962a2d9)


[WebSite](https://qwoyn.studio/)\
[GitHub](https://github.com/cosmic-horizon/QWOYN.git)
=
[EXPLORER](https://explorer.stavr.tech/Qwoyn-Mainnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 150GB    |


# 1) Auto_install script
```python
wget -O qwoynm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cosmic_Horizon/qwoynm && chmod +x qwoynm && ./qwoynm
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

# Build 25.01.24
```python
cd $HOME
git clone https://github.com/cosmic-horizon/QWOYN.git
cd QWOYN
git checkout v5.4.1
make install
```
*******🟢UPDATE🟢******* 25.01.24
```python
cd $HOME/QWOYN && git pull
git checkout v5.4.1
make install
qwoynd version --long | grep -e commit -e version
#version: 5.4.1
#commit: 9a082a7f0839b6d324ae9d65824569f8b1787593
sudo systemctl restart qwoynd && sudo journalctl -u qwoynd -f -o cat
```

`qwoynd version --long`
- version: 5.4.1
- commit: 9a082a7f0839b6d324ae9d65824569f8b1787593

```python
qwoynd init STAVR_guide --chain-id qwoyn-1
qwoynd config chain-id qwoyn-1
```    

## Create/recover wallet
```python
qwoynd keys add <walletname>
  OR
qwoynd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.qwoynd/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cosmic_Horizon/genesis.json"
```
`sha256sum $HOME/.qwoynd/config/genesis.json`
+ 9dd858eb43729f8b53a919212eec25df6a34d02b9c45079d3341923b1f3e67bd

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025uqwoyn\"/;" ~/.qwoynd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.qwoynd/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.qwoynd/config/config.toml
seeds="400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@qwoyn.rpc.kjnodes.com:16359"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.qwoynd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.qwoynd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.qwoynd/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.qwoynd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.qwoynd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.qwoynd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.qwoynd/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.qwoynd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.qwoynd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cosmic_Horizon/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/qwoynd.service > /dev/null <<EOF
[Unit]
Description=qwoynd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which qwoynd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Qwoyn Mainnet
```python
SNAP_RPC=https://qwoyn.rpc.m.stavr.tech:443
SEEDS=becf65550dd8453e36393cb2b97ca4e2494b2460@qwoyn.peer.stavr.tech:11206
cp $HOME/.qwoynd/data/priv_validator_state.json $HOME/.qwoynd/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.qwoynd/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.qwoynd/config/config.toml
qwoynd tendermint unsafe-reset-all --home $HOME/.qwoynd --keep-addr-book
mv $HOME/.qwoynd/priv_validator_state.json.backup $HOME/.qwoynd/data/priv_validator_state.json
sudo systemctl restart qwoynd && journalctl -u qwoynd -f -o cat
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop qwoynd
cp $HOME/.qwoynd/data/priv_validator_state.json $HOME/.qwoynd/priv_validator_state.json.backup
rm -rf $HOME/.qwoynd/data
curl -o - -L http://qwoyn.snapshot.stavr.tech:2/qwoynd/qwoynd-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.qwoynd --strip-components 2
mv $HOME/.qwoynd/priv_validator_state.json.backup $HOME/.qwoynd/data/priv_validator_state.json
wget -O $HOME/.qwoynd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cosmic_Horizon/addrbook.json"
sudo systemctl restart qwoynd && journalctl -u qwoynd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable qwoynd
sudo systemctl restart qwoynd && sudo journalctl -u qwoynd -f -o cat
```

### Create validator
```python
qwoynd tx staking create-validator \
  --amount=1000000uqwoyn \
  --pubkey=$(qwoynd tendermint show-validator) \
  --moniker="STAVRguide" \
  --chain-id="qwoyn-1" \
  --commission-rate="0.01" \
  --commission-max-rate="0.25" \
  --commission-max-change-rate="0.2" \
  --details="" \
  --identity="" \
  --website="" \
  --min-self-delegation="1" \
  --fees 2000uqwoyn \
  --from=wallet \
  --fees="1000uqwoyn" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Qwoyn)
=

## Delete node
```python
sudo systemctl stop qwoynd
sudo systemctl disable qwoynd
rm /etc/systemd/system/qwoynd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf QWOYN
rm -rf .qwoynd
rm -rf $(which qwoynd)
```
#
### Sync Info
```python
qwoynd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
qwoynd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u qwoynd -f -o cat
```
### Check Balance
```python
qwoynd query bank balances qwoynd...addressjkl1yjgn7z09ua9vms259j
```
