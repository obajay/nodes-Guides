# Sao Testnet guide

![Sao](https://user-images.githubusercontent.com/44331529/223452452-434790b5-0079-4402-b6f5-9ddc878ac826.png)

[WebSite](https://www.sao.network/#/)\
[GitHub]( https://github.com/SaoNetwork)
=
[EXPLORER](https://explorer.stavr.tech/sao-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O sao https://raw.githubusercontent.com/obajay/nodes-Guides/main/Sao/sao && chmod +x sao && ./sao
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19.4
```python
ver="1.19.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 26.04.23
```python
cd $HOME
git clone https://github.com/SaoNetwork/sao-consensus.git
cd sao-consensus
git checkout v0.1.4
make install
```
*******🟢UPDATE🟢******* 26.04.23
```python
cd $HOME/sao-consensus
git fetch
git checkout v0.1.4
make install
saod version --long | grep -e commit -e version
#version: 0.1.4
#commit: 8c057b2abb2041ba242f1977c4ea3047461496ee
sudo systemctl restart saod && sudo journalctl -u saod -f -o cat

```

`saod version --long`
- version: 0.1.4
- commit: 8c057b2abb2041ba242f1977c4ea3047461496ee

```python
saod init STAVRguide --chain-id sao-testnet1
saod config chain-id sao-testnet1
```    

## Create/recover wallet
```python
saod keys add <walletname>
  OR
saod keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.sao/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Sao/genesis.json"

```
`sha256sum $HOME/.sao/config/genesis.json`
+ 4df3995bbe58f769b5f312e3bcecfcd4779fc78fdd94519127c6b59a6da89d08

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0sao\"/;" ~/.sao/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sao/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sao/config/config.toml
peers="099fae8829071292f6b1cfaa2b5d637da4aac1b9@203.23.128.181:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sao/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sao/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.sao/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.sao/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sao/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sao/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.sao/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Sao/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/saod.service > /dev/null <<EOF
[Unit]
Description=sao
After=network-online.target

[Service]
User=$USER
ExecStart=$(which saod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Sao Testnet
```python
SNAP_RPC=http://sao.rpc.t.stavr.tech:1077
peers="006e207a3f235a28bc0815001b76ee385ee4bda3@sao.peers.stavr.tech:1076"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sao/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.sao/config/config.toml
saod tendermint unsafe-reset-all --home /root/.sao --keep-addr-book
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.sao/config/app.toml
sudo systemctl restart saod && journalctl -u saod -f -o cat
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop saod
cp $HOME/.sao/data/priv_validator_state.json $HOME/.sao/priv_validator_state.json.backup
rm -rf $HOME/.sao/data
curl -o - -L http://sao.snapshot.stavr.tech:1025/sao/sao-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.sao --strip-components 2
mv $HOME/.sao/priv_validator_state.json.backup $HOME/.sao/data/priv_validator_state.json
wget -O $HOME/.sao/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Sao/addrbook.json"
sudo systemctl restart saod && journalctl -u saod -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable saod
sudo systemctl restart saod && sudo journalctl -u saod -f -o cat
```

### Create validator
```python
saod tx staking create-validator \
  --amount=1000000sao \
  --pubkey=$(saod tendermint show-validator) \
  --moniker="STAVRguide" \
  --details="" \
  --identity="" \
  --website="" \
  --chain-id="sao-testnet1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.50" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=wallet \
  --node "tcp://localhost:1077" -y
```

## Delete node
```python
sudo systemctl stop saod && \
sudo systemctl disable saod && \
rm /etc/systemd/system/saod.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf sao-consensus && \
rm -rf .sao && \
rm -rf $(which saod)
```
#
### Sync Info
```python
saod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
saod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u saod -f -o cat
```
### Check Balance
```python
saod query bank balances saod...addressjkl1yjgn7z09ua9vms259j
```
