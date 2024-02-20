<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 651.920 | 0.060374 | 39.359018 |

<!-- END_TABLE -->



































[🔥OUR VALIDATOR🔥](https://restake.app/firmachain/firmavaloper1xf9fakfwjhq6enq27w3zrpwzav3h2savcs6clh)
=

# FirmaChain Mainnet guide
![firma](https://github.com/obajay/nodes-Guides/assets/44331529/e0755438-89cb-4f1e-bed3-5ca72365b61d)

[WebSite](https://firmachain.org/)\
[GitHub](https://github.com/FirmaChain)
=
[EXPLORER](https://explorer.stavr.tech/Firmachain-M)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O firmam https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Firmachain/firmam && chmod +x firmam && ./firmam
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

# Build 20.12.22
```python
cd $HOME
git clone https://github.com/FirmaChain/firmachain.git
cd firmachain
git checkout v0.3.5-patch
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`firmachaind version --long | head`
- version: 0.3.5-patch
- commit: ac86456416cf590b081ddbe4ff8268e38c21adbd

```python
firmachaind init STAVRguide --chain-id colosseum-1
firmachaind config chain-id colosseum-1
```    

## Create/recover wallet
```python
firmachaind keys add <walletname>
            OR
firmachaind keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.firmachain/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Firmachain/genesis.json"

```
`sha256sum $HOME/.firmachain/config/genesis.json`
+ d03edc3362a677f4a0c2f605c7f848f9516fd4f55432fda63625398c52419954

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.1ufct\"/;" ~/.firmachain/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.firmachain/config/config.toml
peers="a94f70e215a429f4b479ff463183703c0b315d01@144.76.97.251:26116"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.firmachain/config/config.toml
seeds="f89dcc15241e30323ae6f491011779d53f9a5487@mainnet-seed1.firmachain.dev:26656,04cce0da4cf5ceb5ffc04d158faddfc5dc419154@mainnet-seed2.firmachain.dev:26656,940977bdc070422b3a62e4985f2fe79b7ee737f7@mainnet-seed3.firmachain.dev:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.firmachain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.firmachain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.firmachain/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.firmachain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.firmachain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.firmachain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.firmachain/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.firmachain/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.firmachain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Firmachain/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/firmachaind.service > /dev/null <<EOF
[Unit]
Description=firmachaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which firmachaind) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Firmachain Mainnet
```python
SNAP_RPC=https://firma.rpc.m.stavr.tech:443
SEEDS=35b9e0a0818d2c5e9ef187984872c0ad2dbd447c@firma.peer.stavr.tech:1036
cp $HOME/.firmachain/data/priv_validator_state.json $HOME/.firmachain/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.firmachain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.firmachain/config/config.toml
firmachaind tendermint unsafe-reset-all --home $HOME/.firmachain --keep-addr-book
mv $HOME/.firmachain/priv_validator_state.json.backup $HOME/.firmachain/data/priv_validator_state.json
curl -o - -L http://firma.wasm.stavr.tech:12/wasm-firma.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.firmachain --strip-components 2
sudo systemctl restart firmachaind && journalctl -u firmachaind -f -o cat
```
# SnapShot Mainnet (~0.9 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop firmachaind
cp $HOME/.firmachain/data/priv_validator_state.json $HOME/.firmachain/priv_validator_state.json.backup
rm -rf $HOME/.firmachain/data
curl -o - -L http://firma.snapshot.stavr.tech:6/firmachain/firmachain-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.firmachain --strip-components 2
curl -o - -L http://firma.wasm.stavr.tech:12/wasm-firma.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.firmachain --strip-components 2
mv $HOME/.firmachain/priv_validator_state.json.backup $HOME/.firmachain/data/priv_validator_state.json
wget -O $HOME/.firmachain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Firmachain/addrbook.json"
sudo systemctl restart firmachaind && journalctl -u firmachaind -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable firmachaind
sudo systemctl restart firmachaind && sudo journalctl -u firmachaind -f -o cat
```

### Create validator
```python
firmachaind tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000ufct \
--pubkey $(firmachaind tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id colosseum-1 \
--fees="20000ufct" \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Firmachain)
=

## Delete node
```python
sudo systemctl stop firmachaind
sudo systemctl disable firmachaind
rm /etc/systemd/system/firmachaind.service
sudo systemctl daemon-reload
cd $HOME
rm -rf firmachain
rm -rf .firmachain
rm -rf $(which firmachaind)
```
#
### Sync Info
```python
firmachaind status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
firmachaind status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u firmachaind -f -o cat
```
### Check Balance
```python
firmachaind query bank balances firma...addressjkl1yjgn7z09ua9vms259j
```
