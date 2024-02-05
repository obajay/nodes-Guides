# Dymension Mainnet guide

![dymension](https://user-images.githubusercontent.com/44331529/216242184-e602001a-8794-495a-81fc-b0d10589963e.png)


[WebSite](https://dymension.xyz/) \
[GitHub](https://github.com/dymensionxyz/testnets)
=
[EXPLORER](https://explorer.stavr.tech/Dymension-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O dymm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Dymension/dymm && chmod +x dymm && ./dymm
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

# Build 06.02.24
```python
cd $HOME
git clone https://github.com/dymensionxyz/dymension.git
cd dymension
git checkout v3.0.0
make install

```
*******🟢UPDATE🟢******* 00.00.24
```python
SOOOON
```

`dymd version --long | grep -e commit -e version`
- version: v3.0.0
- commit: c3294dc8d2dce1aa8efbc967b1dfd3b0e965b095

```python
dymd init STAVR_guide --chain-id=dymension_1100-1
dymd config chain-id dymension_1100-1
```    

## Create/recover wallet
```python
dymd keys add <walletname>
dymd keys add <walletname> --recover
```

## Download Genesis
```python
wget https://github.com/dymensionxyz/networks/raw/main/mainnet/dymension/genesis.json -O $HOME/.dymension/config/genesis.json
```
`sha256sum $HOME/.dymension/config/genesis.json`
+ 44a4440d7515cd3b7245bc8ed0ccb1e9ecadd8f24da5508f325f9df0509f916b

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"20000000000adym\"/;" ~/.dymension/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.dymension/config/config.toml
peers="a53c0884c5225f13a99e0af9ef04cd50facee668@84.203.117.234:26691,879aca1f688346ca7a3901aa1e9fc62f48112f01@65.109.124.111:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml
seeds="284313184f63d9f06b218a67a0e2de126b64258d@seeds.silknodes.io:25155,2031373ae11f2ce79a5e61fb15308c860559cc64@tenderseed.ccvalidators.com:29202,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:20556,b02a7c844834bc06ffa7ed3a4e51e3c37462872b@seeds.bh.rocks:40656,baeaf56bca434dd5e0dea3900a9368110b4971f8@seed.hoodrun.io:26656,400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@dymension.rpc.kjnodes.com:14659"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.dymension/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.dymension/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.dymension/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.dymension/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" &&
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.dymension/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Dymension/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/dymd.service > /dev/null <<EOF
[Unit]
Description=dymd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which dymd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Dymension Mainnet
```python
SNAP_RPC=https://dym.rpc.m.stavr.tech:443
peers="e0d84deab2d0fd85f447c5c417fecbbdba584be0@dymension-m.peer.stavr.tech:17086"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.dymension/config/config.toml
dymd tendermint unsafe-reset-all --home /root/.dymension
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Dymension/addrbook.json"
systemctl restart dymd && journalctl -u dymd -f -o cat

```
# SnapShot Mainnet updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop dymd
cp $HOME/.dymension/data/priv_validator_state.json $HOME/.dymension/priv_validator_state.json.backup
rm -rf $HOME/.dymension/data
curl -o - -L https://dymension-m.snapshot.stavr.tech/dymension-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.dymension --strip-components 2
mv $HOME/.dymension/priv_validator_state.json.backup $HOME/.dymension/data/priv_validator_state.json
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Dymension/addrbook.json"
sudo systemctl restart dymd && journalctl -u dymd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable dymd
sudo systemctl restart dymd && sudo journalctl -u dymd -f -o cat
```

### Create validator
```python
dymd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.2 \
--min-self-delegation "1000000" \
--amount 1000000000000000000adym \
--pubkey $(dymd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id="dymension_1100-1" \
--identity="" \
--website="" \
--details="" \
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Dymension)
=

## Delete node
```python
sudo systemctl stop dymd
sudo systemctl disable dymd
rm /etc/systemd/system/dymd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf dymension
rm -rf .dymension
rm -rf $(which dymd)
```
#
### Sync Info
```python
dymd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
dymd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u dymd -f -o cat
```
### Check Balance
```python
dymd query bank balances dym...addressjkl1yjgn7z09ua9vms259j
```
