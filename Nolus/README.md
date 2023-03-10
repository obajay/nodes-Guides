# Nolus Testnet guide

![nolus](https://user-images.githubusercontent.com/44331529/209941074-e137b77b-3d27-4e3d-a338-a773460b8a6c.png)


[WebSite](https://nolus.io) \
[GitHub](https://github.com/Nolus-Protocol/nolus-core)
=
[EXPLORER 1](https://explorer.stavr.tech/nolus-testnet/staking) \
[EXPLORER 2](https://exp.utsa.tech/nolus-test)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O nol https://raw.githubusercontent.com/obajay/nodes-Guides/main/Nolus/nol && chmod +x nol && ./nol
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

# Build 10.03.23
```python
cd $HOME
git clone https://github.com/Nolus-Protocol/nolus-core
cd nolus-core
git checkout v0.2.1-testnet
make install
```
*******🟢UPDATE🟢******* 10.03.23
```python
cd $HOME/nolus-core
git fetch --all
git checkout v0.2.1-testnet
make install
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.nolus/config/app.toml
sudo systemctl restart nolusd && journalctl -u nolusd -f -o cat
```


`nolusd version --long`
- version: 0.2.1-testnet
- commit: b33e872e7411d86549a4a8ac1ffbf9b1f5190efa

```python
nolusd init STAVRguide --chain-id nolus-rila
```    

## Create/recover wallet
```python
nolusd keys add <walletname>
nolusd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.nolus/config/genesis.json "https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/genesis.json"
```
`sha256sum $HOME/.nolus/config/genesis.json`
+ d22ea6488afe58478c54afeb2d6b5a45622c797dfd75c91a8653eb1f094173c5

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0unls\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.nolus/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.nolus/config/config.toml
peers="56cee116ac477689df3b4d86cea5e49cfb450dda@54.246.232.38:26656,56f14005119e17ffb4ef3091886e6f7efd375bfd@34.241.107.0:26656,7f26067679b4323496319fda007a279b52387d77@63.35.222.83:26656,7f4a1876560d807bb049b2e0d0aa4c60cc83aa0a@63.32.88.49:26656,3889ba7efc588b6ec6bdef55a7295f3dd559ebd7@3.249.209.26:26656,de7b54f988a5d086656dcb588f079eb7367f6033@34.244.137.169:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.nolus/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.nolus/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.nolus/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.nolus/config/config.toml

```
### Pruning
```python
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.nolus/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.nolus/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.nolus/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Nolus/addrbook.json"
```
## StateSync
```python
SNAP_RPC=http://nolus.rpc.t.stavr.tech:1177
peers="5bf83be8dfe52fe2c204300f1e9b1449487ce5af@nolus.peer.stavr.tech:1176"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.nolus/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 300)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.nolus/config/config.toml
nolusd tendermint unsafe-reset-all --home /root/.nolus --keep-addr-book
systemctl restart nolusd && journalctl -u nolusd -f -o cat
```
## SnapShot (~0.1 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop nolusd
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
rm -rf $HOME/.nolus/data
curl -o - -L http://nolus.snapshot.stavr.tech:1010/nolus/nolus-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.nolus --strip-components 2
curl -o - -L http://nolus.wasm.stavr.tech:1003/wasm-nolus.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.nolus --strip-components 2
mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
wget -O $HOME/.nolus/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Nolus/addrbook.json"
sudo systemctl restart nolusd && journalctl -u nolusd -f -o cat
```

# Create a service file
```python
tee /etc/systemd/system/nolusd.service > /dev/null <<EOF
[Unit]
Description=nolusd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nolusd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable nolusd
sudo systemctl restart nolusd && sudo journalctl -u nolusd -f -o cat
```

### Create validator
```python
nolusd tx staking create-validator \
  --amount 1000000unls \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(nolusd tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id nolus-rila \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop nolusd && \
sudo systemctl disable nolusd && \
rm /etc/systemd/system/nolusd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf nolus-core && \
rm -rf .nolus && \
rm -rf $(which nolusd)
```
#
### Sync Info
```python
nolusd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
nolusd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u nolusd -f -o cat
```
### Check Balance
```python
nolusd query bank balances nolus...address1yjgn7z09ua9vms259j
```
