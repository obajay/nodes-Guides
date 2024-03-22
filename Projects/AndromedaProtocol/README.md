<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 159518.073 | 1.21 | 193016.868931 |

<!-- END_TABLE -->







[🔥OUR VALIDATOR🔥](https://restake.app/andromeda/andrvaloper1xsm4dvmh5txrcteyv3yq5z8pm58mu35365tpzq)
=

# AndromedaProtocol MAINNET guide

![andro](https://user-images.githubusercontent.com/44331529/218533292-7846187a-3897-46ad-9cc8-f17cc18253d3.png)

[WebSite](https://andromedaprotocol.io/)\
[GitHub](https://github.com/andromedaprotocol)
=
[EXPLORER](https://explorer.stavr.tech/Andromeda-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB    |


# 1) Auto_install script
```python
wget -O androm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/AndromedaProtocol/androm && chmod +x androm && ./androm
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

# Build 24.11.23
```python
cd $HOME
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad
git checkout andromeda-1-v0.1.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`andromedad version --long`
- version: andromeda-1-v0.1.0
- commit: a72f010f8e3f9db183da0ddaf4ef65069b690981

```python
andromedad init STAVR_guide --chain-id andromeda-1
andromedad config chain-id andromeda-1
```    

## Create/recover wallet
```python
andromedad keys add <walletname>
  OR
andromedad keys add <walletname> --recover
```

## Download Genesis
```python
wget -L -O $HOME/.andromeda/config/genesis.json https://github.com/andromedaprotocol/mainnet/blob/release/updated_genesis.json
```
`sha256sum $HOME/.andromeda/config/genesis.json`
+ d98f6c7e9c6eb87d59eac0b60883131d637088415aad91f3fed58488db830452

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uandr\"/;" ~/.andromeda/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.andromeda/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.andromeda/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.andromeda/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.andromeda/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.andromeda/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.andromeda/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.andromeda/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.andromeda/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.andromeda/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.andromeda/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.andromeda/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.andromeda/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/AndromedaProtocol/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/andromedad.service > /dev/null <<EOF
[Unit]
Description=andromedad
After=network-online.target

[Service]
User=$USER
ExecStart=$(which andromedad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Andromedad Mainnet
```python
SNAP_RPC=https://andro.rpc.m.stavr.tech:443
peers=e4c2267b90c7cfbb45090ab7647dc01df97f58f9@andromeda-m.peer.stavr.tech:4376
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.andromeda/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.andromeda/config/config.toml
andromedad tendermint unsafe-reset-all --home $HOME/.andromeda
systemctl restart andromedad && journalctl -u andromedad -f -o cat

```
# SnapShot Mainnet (~0.5GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop andromedad
cp $HOME/.andromeda/data/priv_validator_state.json $HOME/.andromeda/priv_validator_state.json.backup
rm -rf $HOME/.andromeda/data
curl -o - -L http://andro.snapshot.stavr.tech:1030/andromeda/andromeda-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.andromeda --strip-components 2
curl -o - -L http://andro.wasm.stavr.tech:1017/wasm-andromeda.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.andromeda --strip-components 2
mv $HOME/.andromeda/priv_validator_state.json.backup $HOME/.andromeda/data/priv_validator_state.json
wget -O $HOME/.andromeda/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/AndromedaProtocol/addrbook.json"
sudo systemctl restart andromedad && journalctl -u andromedad -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable andromedad
sudo systemctl restart andromedad && sudo journalctl -u andromedad -f -o cat
```

### Create validator
```python
andromedad tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000uandr \
--pubkey $(andromedad tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id andromeda-1 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/AndromedaProtocol)
=



## Delete node
```python
sudo systemctl stop andromedad && \
sudo systemctl disable andromedad && \
rm /etc/systemd/system/andromedad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf andromedad && \
rm -rf .andromeda && \
rm -rf $(which andromedad)
```
#
### Sync Info
```python
andromedad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
andromedad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u andromedad -f -o cat
```
### Check Balance
```python
andromedad query bank balances andr...addressjkl1yjgn7z09ua9vms259j
```
