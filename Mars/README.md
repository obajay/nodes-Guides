# Mars Testnet guide

![mars](https://user-images.githubusercontent.com/44331529/212458394-28b38996-0371-4841-aa51-1d2a828453ee.png)


[WebSite](https://marsprotocol.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/mars-testnet/staking) \
[EXPLORER 2](https://explorer.nodestake.top/blockx-testnet/staking) \
[EXPLORER 3](https://testnet-explorer.marsprotocol.io/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O marss https://raw.githubusercontent.com/obajay/nodes-Guides/main/Mars/marss && chmod +x marss && ./marss
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

# Build 14.01.23
```python
git clone https://github.com/mars-protocol/hub mars
cd mars
git checkout v1.0.0-rc7
make install
```
`marsd version --long`
- version: 1.0.0-rc7
- commit: 4cc5d7e7aa4e92a117e283b08221eb4c285bf141

```python
marsd init STAVRguide --chain-id ares-1
```    

## Create/recover wallet
```python
marsd keys add <walletname>
marsd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O ~/.mars/config/genesis.json https://raw.githubusercontent.com/mars-protocol/networks/main/ares-1/genesis.json
```

`sha256sum $HOME/.mars/config/genesis.json`
+ a600fc081fabdc6dccf0f11591b0c88b62dee11a21dfa23834ab825351bf8e2f

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0umars\"/" $HOME/.mars/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.mars/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.mars/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mars/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:18556"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.mars/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.mars/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.mars/config/config.toml
```

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.mars/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.mars/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.mars/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.mars/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mars/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.mars/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Mars/addrbook.json"
```
## StateSync
```python
SNAP_RPC=http://mars.rpc.t.stavr.tech:190/
peers="b42f07453d051f65978c22b8047feb9d2e634aff@mars.peer.stavr.tech:181"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.mars/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 300)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.mars/config/config.toml
marsd tendermint unsafe-reset-all --home /root/.mars --keep-addr-book
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.mars/config/app.toml
systemctl restart marsd && journalctl -u marsd -f -o cat
```
## SnapShot (~0.2 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop marsd
cp $HOME/.mars/data/priv_validator_state.json $HOME/.mars/priv_validator_state.json.backup
rm -rf $HOME/.mars/data
curl -o - -L http://mars.snapshot.stavr.tech:1012/mars/mars-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.mars --strip-components 2
mv $HOME/.mars/priv_validator_state.json.backup $HOME/.mars/data/priv_validator_state.json
wget -O $HOME/.mars/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Mars/addrbook.json"
sudo systemctl restart marsd && journalctl -u marsd -f -o cat
```

# Create a service file
```python
sudo tee /etc/systemd/system/marsd.service > /dev/null <<EOF
[Unit]
Description=sge
After=network-online.target

[Service]
User=$USER
ExecStart=$(which marsd) start
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
sudo systemctl enable marsd
sudo systemctl restart marsd && sudo journalctl -u marsd -f -o cat
```

### Create validator
```python
marsd tx staking create-validator \
  --amount 1000000umars \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(marsd tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id ares-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop marsd && \
sudo systemctl disable marsd && \
rm /etc/systemd/system/marsd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf mars && \
rm -rf .mars && \
rm -rf $(which marsd)
```
#
### Sync Info
```python
marsd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
marsd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u marsd -f -o cat
```
### Check Balance
```python
marsd query bank balances mars...address1yjgn7z09ua9vms259j
```
