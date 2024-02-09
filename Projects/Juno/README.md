<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 7566.8 | 0.3545 | 2682.456548 |

<!-- END_TABLE -->






[🔥OUR VALIDATOR🔥](https://restake.app/juno/junovaloper177svs8gyz28pupaj6amgls6rahcmwmwq06fukv)
=

# Juno Mainnet guide
![juno](https://user-images.githubusercontent.com/44331529/222464046-d33e1e1f-ac42-4f9c-8c34-7f978772d5ed.png)

[WebSite](https://www.junonetwork.io/)\
[GitHub](https://github.com/CosmosContracts)
=
[EXPLORER 1](https://explorer.stavr.tech/Juno/staking) \
[EXPLORER 2](https://ping.pub/juno/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  16| 32GB | 250GB    |


# 1) Auto_install script
```python
wget -O jun https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Juno/jun && chmod +x jun && ./jun
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19 
```python
ver="1.19"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 08.02.24
```python
cd $HOME
git clone https://github.com/CosmosContracts/juno juno
cd juno
git checkout v20.0.0
make install
```
*******🟢UPDATE🟢******* 08.02.24
```python
cd $HOME/juno
git checkout v20.0.0
make install
junod version --long | grep -e version -e commit
#version: v20.0.0
#commit: 6f27cf8883fbbbcc2ff9f6ae6662816fae616278
sudo systemctl restart junod && sudo journalctl -u junod -f -o cat
```

`junod version --long`
- version: v20.0.0
- commit: 6f27cf8883fbbbcc2ff9f6ae6662816fae616278

```python
junod init STAVR_guide --chain-id juno-1
junod config chain-id juno-1
```    

## Create/recover wallet
```python
junod keys add <walletname>
  OR
junod keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.juno/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Juno/genesis.json"

```
`sha256sum $HOME/.juno/config/genesis.json`
+ a5c08e53aca0390c45def85a6d16c0e7176bd0026b0a465aff5d1896ec0134a1

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9,0ujuno\"/;" ~/.juno/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.juno/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.juno/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.juno/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:12656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.juno/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.juno/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.juno/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.juno/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.juno/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.juno/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.juno/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.juno/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.juno/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Juno/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/junod.service > /dev/null <<EOF
[Unit]
Description=juno
After=network-online.target

[Service]
User=$USER
ExecStart=$(which junod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Juno Mainnet
```python
SNAP_RPC=https://juno.rpc.m.stavr.tech:443
PEERS="9ec270e1b4cdd38557da9d374ae4333145ee9300@juno.peer.stavr.tech:1066"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.juno/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.juno/config/config.toml
junod tendermint unsafe-reset-all
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.juno/config/app.toml
curl -o - -L https://juno.wasm.stavr.tech/wasm-juno.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.juno/ --strip-components 2
sudo systemctl restart junod && journalctl -u junod -f -o cat
```
# SnapShot Mainnet updated every 7 hours  
```python
cd $HOME
snap install lz4
sudo systemctl stop junod
cp $HOME/.juno/data/priv_validator_state.json $HOME/.juno/priv_validator_state.json.backup
rm -rf $HOME/.juno/data
curl -o - -L https://juno.snapshot.stavr.tech/juno-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.juno --strip-components 2
mv $HOME/.juno/priv_validator_state.json.backup $HOME/.juno/data/priv_validator_state.json
wget -O $HOME/.juno/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Juno/addrbook.json"
sudo systemctl restart junod && journalctl -u junod -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable junod
sudo systemctl restart junod && sudo journalctl -u junod -f -o cat
```

### Create validator
```python
junod tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000ujuno \
--pubkey $(junod tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id juno-1 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Juno)
=

## Delete node
```python
sudo systemctl stop junod
sudo systemctl disable junod
rm /etc/systemd/system/junod.service
sudo systemctl daemon-reload
cd $HOME
rm -rf juno
rm -rf .juno
rm -rf $(which junod)
```
#
### Sync Info
```python
junod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
junod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u junod -f -o cat
```
### Check Balance
```python
junod query bank balances juno...addressjkl1yjgn7z09ua9vms259j
```
