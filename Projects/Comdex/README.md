# Comdex Mainnet guide

![comdex](https://github.com/obajay/nodes-Guides/assets/44331529/85ff40f3-076d-439e-bbcd-d4951c898825)

[WebSite](https://comdex.one/) \
[GitHub](https://github.com/comdex-official/comdex)
=
[EXPLORER 1](https://explorer.stavr.tech/Comdex-Mainnet/) \
[EXPLORER 2](https://ping.pub/comdex/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O comdexm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Comdex/comdexm && chmod +x comdexm && ./comdexm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.20
```python
ver="1.20"
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
cd $HOME && mkdir -p go/bin/
git clone https://github.com/comdex-official/comdex comdex
cd comdex
git checkout v13.4.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`comdex version --long | grep -e commit -e version`
- version: v13.4.0
- commit: 42f2e7393a4dbb739d86dee97a7e7ec3a5769403

```python
comdex init STAVR_guide --chain-id comdex-1
comdex config chain-id comdex-1
```    

## Create/recover wallet
```python
comdex keys add <walletname>
  OR
comdex keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.comdex/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Comdex/genesis.json"
```
`sha256sum $HOME/.comdex/config/genesis.json`
+ 88da94b7346eed173dbdab425272c692ec0f37627c7b8b43ffd3fd0397264273

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025ucmdx\"/;" ~/.comdex/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.comdex/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.comdex/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.comdex/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:13156"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.comdex/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.comdex/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.comdex/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.comdex/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.comdex/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.comdex/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.comdex/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.comdex/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.comdex/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Comdex/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/comdex.service > /dev/null <<EOF
[Unit]
Description=comdex
After=network-online.target

[Service]
User=$USER
ExecStart=$(which comdex) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Comdex Mainnet
```python
SNAP_RPC=https://comdex.rpc.m.stavr.tech:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.comdex/config/config.toml
comdex tendermint unsafe-reset-all
wget -O $HOME/.comdex/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Comdex/addrbook.json"
curl -o - -L https://comdex.wasm.stavr.tech/wasm-comdex.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.comdex --strip-components 2
sudo systemctl restart comdex && sudo journalctl -u comdex -f -o cat
```
# SnapShot Mainnet (~2 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop comdex
cp $HOME/.comdex/data/priv_validator_state.json $HOME/.comdex/priv_validator_state.json.backup
rm -rf $HOME/.comdex/data
curl -o - -L https://comdex.snapshot.stavr.tech/comdex/comdex-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.comdex --strip-components 2
curl -o - -L https://comdex.wasm.stavr.tech/wasm-comdex.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.comdex --strip-components 2
mv $HOME/.comdex/priv_validator_state.json.backup $HOME/.comdex/data/priv_validator_state.json
wget -O $HOME/.comdex/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Comdex/addrbook.json"
sudo systemctl restart comdex && journalctl -u comdex -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable comdex
sudo systemctl restart comdex && sudo journalctl -u comdex -f -o cat
```

### Create validator
```python
comdex tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 100000000ucmdx \
--pubkey $(comdex tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id comdex-1 \
--gas 300000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Comdex)
=


## Delete node
```python
sudo systemctl stop comdex
sudo systemctl disable comdex
rm /etc/systemd/system/comdex.service
sudo systemctl daemon-reload
cd $HOME
rm -rf comdex
rm -rf .comdex
rm -rf $(which comdex)
```
#
### Sync Info
```python
comdex status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
comdex status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u comdex -f -o cat
```
### Check Balance
```python
comdex query bank balances comdex...addressjkl1yjgn7z09ua9vms259j
```
