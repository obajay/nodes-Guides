<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 1004286.3 | 0.01931138 | 19394.15622353 |

<!-- END_TABLE -->

























































































































[🔥OUR VALIDATOR🔥](https://restake.app/impacthub/ixovaloper1htcnjafe94aqyxfapw2dlqz242g2n2tqp9lulh)
=

# IXO Mainnet guide
![ixo](https://github.com/obajay/nodes-Guides/assets/44331529/3a2a43fc-d642-4696-a0d6-7e2161d44442)

[WebSite]( https://ixo.world/)\
[GitHub](https://github.com/ixofoundation)
=
[EXPLORER](https://explorer.stavr.tech/IXO-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O ixom https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Ixo/ixom && chmod +x ixom && ./ixom
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

# Build 28.08.23
```python
cd $HOME
git clone https://github.com/ixofoundation/ixo-blockchain.git
cd ixo-blockchain
git checkout v2.0.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`ixod version --long | head`
- version: 2.0.0
- commit: f41a093c66cc1bfe39bb276426c3f2610649efc8

```python
ixod init STAVRguide --chain-id ixo-5
ixod config chain-id ixo-5
```    

## Create/recover wallet
```python
ixod keys add <walletname>
            OR
ixod keys add <walletname> --recover
```

## Download Genesis
```python
curl https://anode.team/IXO/main/genesis.json > ~/.ixod/config/genesis.json
```
`sha256sum $HOME/.ixod/config/genesis.json`
+ 2b3440ad0522cfc157be6da3a23dcf0ef4bdf6ab4f643fc0da54c0233bc255d5

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uixo\"/;" ~/.ixod/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.ixod/config/config.toml
peers="a8d9811a2f08b8a6c77e4319097d6fd84520645e@139.84.226.60:26656,f79da5c87e40587c4cfef5d7b7902b6e69ac62bf@188.166.183.216:26656,386277f9c6a0c402889032ff76585d0a2dae7bc5@104.248.1.56:26656,26593e0854848ede80d5cd963dc8a775634e2acc@23.88.69.167:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.ixod/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.ixod/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.ixod/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.ixod/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.ixod/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.ixod/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.ixod/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.ixod/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.ixod/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.ixod/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Ixo/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/ixod.service > /dev/null <<EOF
[Unit]
Description=ixod
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ixod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Ixo Mainnet
```python
SNAP_RPC=https://ixo.rpc.m.stavr.tech:443
SEEDS=d4448c5b10b43d444034533ede7d2e66cbf9e519@ixo.peer.stavr.tech:1016
cp $HOME/.ixod/data/priv_validator_state.json $HOME/.ixod/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.ixod/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.ixod/config/config.toml
ixod tendermint unsafe-reset-all --home $HOME/.ixod --keep-addr-book
mv $HOME/.ixod/priv_validator_state.json.backup $HOME/.ixod/data/priv_validator_state.json
curl -o - -L http://ixo.wasm.stavr.tech:11/wasm-ixod.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.ixod --strip-components 2
sudo systemctl restart ixod && journalctl -u ixod -f -o cat
```
# SnapShot Mainnet (~0.2 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop ixod
cp $HOME/.ixod/data/priv_validator_state.json $HOME/.ixod/priv_validator_state.json.backup
rm -rf $HOME/.ixod/data
curl -o - -L http://ixo.snapshot.stavr.tech:5/ixod/ixod-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.ixod --strip-components 2
curl -o - -L http://ixo.wasm.stavr.tech:11/wasm-ixod.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.ixod --strip-components 2
mv $HOME/.ixod/priv_validator_state.json.backup $HOME/.ixod/data/priv_validator_state.json
wget -O $HOME/.ixod/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Ixo/addrbook.json"
sudo systemctl restart ixod && journalctl -u ixod -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable ixod
sudo systemctl restart ixod && sudo journalctl -u ixod -f -o cat
```

### Create validator
```python
ixod tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000uixo \
--pubkey $(ixod tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id ixo-5 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Ixo)
=

## Delete node
```python
sudo systemctl stop ixod
sudo systemctl disable ixod
rm /etc/systemd/system/ixod.service
sudo systemctl daemon-reload
cd $HOME
rm -rf ixo-blockchain
rm -rf .ixod
rm -rf $(which ixod)
```
#
### Sync Info
```python
ixod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
ixod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u ixod -f -o cat
```
### Check Balance
```python
ixod query bank balances ixo...addressjkl1yjgn7z09ua9vms259j
```
