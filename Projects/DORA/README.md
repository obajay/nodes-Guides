# Dora Factory Testnet guide


![dora](https://github.com/obajay/nodes-Guides/assets/44331529/024491f0-3b7b-4652-a1a0-1784500818e0)

[WebSite](https://dorafactory.org/)\
[GitHub](https://github.com/DoraFactory/doravota.git)
=
[EXPLORER](https://explorer.stavr.tech/Dora-Testnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   8|  16GB | 250GB    |


# 1) Auto_install script
```python
wget -O dorat https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/DORA/dorat && chmod +x dorat && ./dorat
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

# Build 21.12.23
```python
cd $HOME
git clone https://github.com/DoraFactory/doravota.git
cd doravota
git checkout 0.2.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`dorad version --long | grep -e version -e commit`
- version: 0.2.0
- commit: 

```python
dorad init STAVR_guide --chain-id vota-vk
dorad config chain-id vota-vk
```    

## Create/recover wallet
```python
dorad keys add <walletname>
  OR
dorad keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.dora/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/DORA/genesis.json"

```
`sha256sum $HOME/.dora/config/genesis.json`
+ dc3a959080f9291e59b92528e7e2ee95961c6609562fda76c8db1946dd3c8fef

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"100000000000peaka\"/;" ~/.dora/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.dora/config/config.toml
peers="2011dfc8a30cf14ffb7207764c2549c210d4c598@136.243.69.100:56096,cdf3cb078183967cff3a638713384de2c5594ba3@65.108.72.253:42656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dora/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.dora/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.dora/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.dora/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.dora/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.dora/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.dora/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.dora/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.dora/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.dora/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/DORA/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/dorad.service > /dev/null <<EOF
[Unit]
Description=dorad
After=network-online.target

[Service]
User=$USER
ExecStart=$(which dorad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Dora Testnet
```python
SNAP_RPC=https://dora.rpc.t.stavr.tech:443
peers="3c21389d10b9499df09a7eb36afa8e748433c286@dora-t.peer.stavr.tech:32046"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dora/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.dora/config/config.toml
dorad tendermint unsafe-reset-all --home /root/.dora
systemctl restart dorad && journalctl -u dorad -f -o cat
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop dorad
cp $HOME/.dora/data/priv_validator_state.json $HOME/.dora/priv_validator_state.json.backup
rm -rf $HOME/.dora/data
curl -o - -L https://dorat.snapshot.stavr.tech/dora-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.dora --strip-components 2
curl -o - -L http://dorat.wasm.stavr.tech:1103/wasm-dora.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.dora --strip-components 2
mv $HOME/.dora/priv_validator_state.json.backup $HOME/.dora/data/priv_validator_state.json
wget -O $HOME/.dora/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/DORA/addrbook.json"
sudo systemctl restart dorad && journalctl -u dorad -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable dorad
sudo systemctl restart dorad && sudo journalctl -u dorad -f -o cat
```

### Create validator
```python
dorad tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000000000000000peaka \
--pubkey $(dorad tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id vota-vk \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Dora)
=

## Delete node
```python
sudo systemctl stop dorad
sudo systemctl disable dorad
rm /etc/systemd/system/dorad.service
sudo systemctl daemon-reload
cd $HOME
rm -rf doravota
rm -rf .dora
rm -rf $(which dorad)
```
#
### Sync Info
```python
dorad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
dorad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u dorad -f -o cat
```
### Check Balance
```python
dorad query bank balances dora...addressjkl1yjgn7z09ua9vms259j
```

