# Composable Testnet-5 guide

![compo](https://github.com/obajay/nodes-Guides/assets/44331529/49502f93-cb03-461e-b788-78a391456f72)

[WebSite](https://www.composable.finance/)\
[GitHub](https://github.com/notional-labs/composable-networks)
=
[EXPLORER](https://explorer.stavr.tech/Composable-Testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet-3   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O compost4 https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Composable/Testnet-5/compost4 && chmod +x compost4 && ./compost4
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
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

# Build 24.01.24
```python
cd $HOME
git clone https://github.com/notional-labs/composable-centauri
cd composable-centauri
git checkout v6.4.x-staking-distribution-patch-tag
make build
cd bin
mv centaurid $HOME/go/bin/centaurid
```

*******🟢UPDATE🟢******* 24.01.24
```python
cd $HOME/composable-centauri
git pull
git checkout v6.4.x-staking-distribution-patch-tag
make install
centaurid version --long | grep -e commit -e version
#version: v6.4.x-staking-distribution-patch-tag
#commit: f1eee5402bf54f4069919d1426d958be46e584a0
sudo systemctl restart centaurid && sudo journalctl -u centaurid -f -o cat
```

`centaurid version --long`
- version: v6.4.x-staking-distribution-patch-tag
- commit: f1eee5402bf54f4069919d1426d958be46e584a0

```python
centaurid init STAVR_guide --chain-id banksy-testnet-5
centaurid config chain-id banksy-testnet-5
```    

## Create/recover wallet
```python
centaurid keys add <walletname>
  OR
centaurid keys add <walletname> --recover
```

## Download Genesis
```python
curl -Ls https://raw.githubusercontent.com/notional-labs/composable-networks/main/banksy-testnet-5/genesis.json > ~/.banksy/config/genesis.json
```
`sha256sum $HOME/.banksy/config/genesis.json`
+ 5fbcc1a2d40de313977d8861e3f775d290ec013d7a63cc73432139950f0c4ce7

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.1ppica\"/;" ~/.banksy/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.banksy/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.banksy/config/config.toml
seeds="6e8a56df9b9c52a730dd780172fc135a96a9feda@65.109.26.223:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.banksy/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.banksy/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.banksy/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.banksy/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.banksy/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.banksy/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Composable/Testnet-5/addrbook.json"
```

# Create a service file
```python
tee /etc/systemd/system/centaurid.service > /dev/null <<EOF
[Unit]
Description=centaurid
After=network-online.target

[Service]
User=$USER
ExecStart=$(which centaurid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Testnet
```python
SOON
```
# SnapShot Testnet (~4GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop layerd
cp $HOME/.banksy/data/priv_validator_state.json $HOME/.banksy/priv_validator_state.json.backup
rm -rf $HOME/.banksy/data
curl -o - -L https://composable-t4.snapshot.stavr.tech/composable/composable-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.banksy --strip-components 2
curl -o - -L https://composable.wasmt4.stavr.tech/wasm-composable.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.banksy --strip-components 2
mv $HOME/.banksy/priv_validator_state.json.backup $HOME/.banksy/data/priv_validator_state.json
wget -O $HOME/.banksy/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Composable/Testnet-5/addrbook.json"
sudo systemctl restart layerd && journalctl -u layerd -f -o cat
```
# WASM FOLDER
```python
curl -o - -L http://composable.wasmT4.stavr.tech:3102/wasm-composable.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.banksy --strip-components 2
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable centaurid
sudo systemctl restart centaurid && sudo journalctl -u centaurid -f -o cat
```

### Create validator
```python
centaurid tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000000000000000ppica \
--pubkey $(layerd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id banksy-testnet-5 \
--fees 2000000000ppica \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop centaurid
sudo systemctl disable centaurid
rm /etc/systemd/system/centaurid.service
sudo systemctl daemon-reload
cd $HOME
rm -rf composable-centauri
rm -rf .banksy
rm -rf $(which centaurid)
```
#
### Sync Info
```python
centaurid status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
centaurid status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u centaurid -f -o cat
```
### Check Balance
```python
layerd query bank balances centaurid...addressjkl1yjgn7z09ua9vms259j
```
