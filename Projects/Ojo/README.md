# Ojo Devnet guide

![ojo](https://user-images.githubusercontent.com/44331529/223457575-2b0b703b-372a-4e33-b725-d703edaf2bf7.png)


[WebSite](https://ojo.network/)\
[GitHub](https://github.com/ojo-network)
=
[EXPLORER](https://explorer.stavr.tech/Ojo-Devnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Devnet    |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O ojjo https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Ojo/ojjo && chmod +x ojjo && ./ojjo
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19
```python
ver="1.19" &&
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" &&
sudo rm -rf /usr/local/go &&
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" &&
rm "go$ver.linux-amd64.tar.gz" &&
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile &&
source $HOME/.bash_profile &&
go version
```

# Build 06.03.23
```python
cd $HOME
git clone https://github.com/ojo-network/ojo
cd ojo
git checkout v0.1.2
make install
```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`ojod version --long`
- version: v2.0.0
- commit: ad5a2377134aa13d7d76575b95613cf8ed12d1e4

```python
ojod init STAVRguide --chain-id ojo-devnet
ojod config chain-id ojo-devnet
```    

## Create/recover wallet
```python
ojod keys add <walletname>
  OR
ojod keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.ojo/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Ojo/genesis.json"

```
`sha256sum $HOME/.ojo/config/genesis.json`
+ 6037d1c1a89110c024fc18143eafe33fee19671b9427a4d4ac9c701f7a3c9309

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uojo\"/;" ~/.ojo/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.ojo/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.ojo/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.ojo/config/config.toml
seeds="9aa8a73ea9364aa3cf7806d4dd25b6aed88d8152@ojo-devnet.seed.mzonder.com:11556"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.ojo/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.ojo/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.ojo/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.ojo/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.ojo/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.ojo/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.ojo/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.ojo/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.ojo/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Ojo/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/ojod.service > /dev/null <<EOF
[Unit]
Description=ojo
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ojod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Ojo Testnet
```python
SNAP_RPC=https://ojo.rpc.t.stavr.tech:443
peers="1f091cf9567c0d72a0f93877007379e0298b8860@ojo.peer.stavr.tech:37096"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.ojo/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.ojo/config/config.toml
ojod tendermint unsafe-reset-all --home /root/.ojo --keep-addr-book
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.ojo/config/app.toml
sudo systemctl restart ojod && journalctl -u ojod -f -o cat
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop ojod
cp $HOME/.ojo/data/priv_validator_state.json $HOME/.ojo/priv_validator_state.json.backup
rm -rf $HOME/.ojo/data
curl -o - -L http://ojo.snapshot.stavr.tech:1026/ojo/ojo-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.ojo --strip-components 2
mv $HOME/.ojo/priv_validator_state.json.backup $HOME/.ojo/data/priv_validator_state.json
wget -O $HOME/.ojo/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Ojo/addrbook.jso"
sudo systemctl restart ojod && journalctl -u ojod -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable ojod
sudo systemctl restart ojod && sudo journalctl -u ojod -f -o cat
```

### Create validator
```python
ojod tx staking create-validator \
  --amount 1000000uojo \
  --from WalletName \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(ojod tendermint show-validator) \
  --moniker "STAVRguide" \
  --chain-id ojo-devnet \
  --identity="" \
  --details="" \
  --website="" -y
```
[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Ojo)
=

## Delete node
```python
sudo systemctl stop ojod && \
sudo systemctl disable ojod && \
rm /etc/systemd/system/ojod.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf ojo && \
rm -rf .ojo && \
rm -rf $(which ojod)
```
#
### Sync Info
```python
ojod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
ojod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u ojod -f -o cat
```
### Check Balance
```python
ojod query bank balances ojo...addressjkl1yjgn7z09ua9vms259j
```

<h1 align="center"> 🔥Install price-feeder🔥</h1> 


```python
# set vars
OJO_MAIN_WALLET="MAIN_WALLET"
OJO_PFD_WALLET="PFD_WALLET"

# add validator wallet
ojod keys add $OJO_MAIN_WALLET --home $OJO_HOME

# add pfd wallet
ojod keys add $OJO_PFD_WALLET --home $OJO_HOME

# save vars
OJO_MAIN_ADDR=$(ojod keys show $OJO_MAIN_WALLET -a --home $OJO_HOME) && echo $OJO_MAIN_ADDR
OJO_VALOPER=$(ojod keys show $OJO_MAIN_WALLET --bech val -a --home $OJO_HOME) && echo $OJO_VALOPER
OJO_PFD_ADDR=$(ojod keys show $OJO_PFD_WALLET -a --home $OJO_HOME) && echo $OJO_PFD_ADDR

# check vars
echo $OJO_MAIN_WALLET,$OJO_PFD_WALLET,$OJO_MAIN_ADDR,$OJO_VALOPER,$OJO_PFD_ADDR | tr "," "\n" | nl 
# output 5 lines

echo "
export OJO_MAIN_WALLET=${OJO_MAIN_WALLET}
export OJO_PFD_WALLET=${OJO_PFD_WALLET}
export OJO_MAIN_ADDR=${OJO_MAIN_ADDR}
export OJO_VALOPER=${OJO_VALOPER}
export OJO_PFD_ADDR=${OJO_PFD_ADDR}
" >> $HOME/.bash_profile

source $HOME/.bash_profile
```

## build price-feeder
```python
cd $HOME
git clone https://github.com/ojo-network/price-feeder
cd price-feeder
git checkout v0.1.1
make install
```
## config price-feeder
```python

OJO_KEYRING="os"
OJO_KEYRING_PASSWORD="devnetdevnet"
OJO_RPC_PORT=26657
OJO_GRPC_PORT=9090

# save vars
echo "
export OJO_KEYRING=${OJO_KEYRING}
export OJO_KEYRING_PASSWORD=${OJO_KEYRING_PASSWORD}
export OJO_RPC_PORT=${OJO_RPC_PORT}
export OJO_GRPC_PORT=${OJO_GRPC_PORT}
" >> $HOME/.bash_profile

source $HOME/.bash_profile

mkdir -p $HOME/price-feeder_config

# get template
wget -O $HOME/price-feeder_config/price-feeder.toml "https://raw.githubusercontent.com/ojo-network/price-feeder/main/price-feeder.example.toml"

# check vars (vars were set above)
echo $OJO_CHAIN,$OJO_HOME,$OJO_VALOPER,$OJO_MAIN_ADDR,$OJO_KEYRING,$OJO_KEYRING_PASSWORD,$OJO_RPC_PORT,$OJO_GRPC_PORT | tr "," "\n" | nl 
# output 8 lines

# add pass option if use "os" or "file" keyring
sed -i '/^dir *=.*/a pass = ""' $HOME/price-feeder_config/price-feeder.toml

# set values
sed -i "s/^address *=.*/address = \"$OJO_MAIN_ADDR\"/;\
s/^chain_id *=.*/chain_id = \"$OJO_CHAIN\"/;\
s/^validator *=.*/validator = \"$OJO_VALOPER\"/;\
s/^backend *=.*/backend = \"$OJO_KEYRING\"/;\
s|^dir *=.*|dir = \"$OJO_HOME\"|;\
s|^pass *=.*|pass = \"$OJO_KEYRING_PASSWORD\"|;\
s|^grpc_endpoint *=.*|grpc_endpoint = \"localhost:${OJO_GRPC_PORT}\"|;\
s|^tmrpc_endpoint *=.*|tmrpc_endpoint = \"http://localhost:${OJO_RPC_PORT}\"|;" $HOME/price-feeder_config/price-feeder.toml
```

## Create service
```python
tee $HOME/price-feeder.service > /dev/null <<EOF
[Unit]
Description=OJO PFD
After=network.target
[Service]
User=$USER
Environment="PRICE_FEEDER_PASS=${OJO_KEYRING_PASSWORD}"
Type=simple
ExecStart=$(which price-feeder) $HOME/price-feeder_config/price-feeder.toml --log-level debug
RestartSec=10
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo mv $HOME/price-feeder.service /etc/systemd/system/

sudo systemctl daemon-reload
sudo systemctl enable price-feeder
sudo systemctl start price-feeder && journalctl -u price-feeder -f -o cat
```
## Fund and delegate price-feeder addr
```python
# fund
ojod tx bank send $OJO_MAIN_ADDR $OJO_PFD_ADDR 1000000uojo --fees 200uojo --home $OJO_HOME
# delegate pfd addr
ojod tx oracle delegate-feed-consent $OJO_MAIN_ADDR $OJO_PFD_ADDR --fees 400uojo --home $OJO_HOME
# edit config 
sed -i "s/^address *=.*/address= \"$OJO_PFD_ADDR\"/" $HOME/price-feeder_config/price-feeder.toml
sudo systemctl restart price-feeder && journalctl -u price-feeder -f -o cat
```
```python
# check ojo status
ojod status --home $OJO_HOME |& jq

# check price-feeder
ojod q oracle slash-window --home $OJO_HOME
ojod q oracle miss-counter $OJO_VALOPER --home $OJO_HOME
```
