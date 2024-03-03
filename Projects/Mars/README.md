<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 784.034 | 0.110423 | 86.575424809 |

<!-- END_TABLE -->



































































































[🔥OUR VALIDATOR🔥](https://restake.app/mars/marsvaloper1pc3ahwcurfedenmmndtk6w4nv7yj08ywfyenqf)
=

<h1 align="center"> 🔥Mars MAINNET guide🔥</h1>

![mars](https://user-images.githubusercontent.com/44331529/212458394-28b38996-0371-4841-aa51-1d2a828453ee.png)

[WebSite](https://marsprotocol.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/Mars-Mainnet) \
[EXPLORER 2](https://explorer.nodestake.top/mars/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB    |


# 1) Auto_install script
```python
wget -O mars1 https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Mars/mars1 && chmod +x mars1 && ./mars1
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

# Build 08.06.23
```python
git clone https://github.com/mars-protocol/hub.git
cd hub
git checkout v1.0.2
make install
```
`marsd version --long`
- version: 1.0.2
- commit: ffbe9c28a6070abbbc7cf4be2f80d6a728715d62

```python
marsd init STAVRguide --chain-id mars-1
marsd config chain-id mars-1
```    

## Create/recover wallet
```python
marsd keys add <walletname>
marsd keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/mars-protocol/networks/main/mars-1/genesis.json -O $HOME/.mars/config/genesis.json
```

`sha256sum $HOME/.mars/config/genesis.json`
+ 7d93bd062e5f6909cb46cf96b64533b821232f8766c6513ced535aa9ad37b425

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0umars\"/" $HOME/.mars/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.mars/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.mars/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mars/config/config.toml
seeds="52de8a7e2ad3da459961f633e50f64bf597c7585@seed.marsprotocol.io:443,d2d2629c8c8a8815f85c58c90f80b94690468c4f@tenderseed.ccvalidators.com:26012"
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
SNAP_RPC=http://mars.rpc.t.stavr.tech:190
peers="d79cd692ab2a4ef1a8be282cb398d6267b871b6d@mars.peer.stavr.tech:181"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.mars/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.mars/config/config.toml
marsd tendermint unsafe-reset-all --home /root/.mars --keep-addr-book
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.mars/config/app.toml
curl -o - -L http://mars.wasm.stavr.tech:1014/wasm-mars.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.mars/data --strip-components 3
systemctl restart marsd && journalctl -u marsd -f -o cat
```
## SnapShot (~0.2 GB)
```python
cd $HOME
apt install lz4
sudo systemctl stop marsd
cp $HOME/.mars/data/priv_validator_state.json $HOME/.mars/priv_validator_state.json.backup
rm -rf $HOME/.mars/data
curl -o - -L http://mars.snapshot.stavr.tech:1012/mars/mars-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.mars --strip-components 2
curl -o - -L http://mars.wasm.stavr.tech:1014/wasm-mars.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.mars/data --strip-components 3
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
  --chain-id mars-1 \
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
### Node Info
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
