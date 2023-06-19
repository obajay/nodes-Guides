# Dymension Testnet guide

![dymension](https://user-images.githubusercontent.com/44331529/216242184-e602001a-8794-495a-81fc-b0d10589963e.png)


[WebSite](https://dymension.xyz/) \
[GitHub](https://github.com/dymensionxyz/testnets)
=
[EXPLORER](https://explorer.stavr.tech/dymension-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O dym https://raw.githubusercontent.com/obajay/nodes-Guides/main/Dymension/dym && chmod +x dym && ./dym
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

# Build 01.02.23
```python
cd $HOME
git clone https://github.com/dymensionxyz/dymension.git --branch v0.2.0-beta
cd dymension
make install

```
*******🟢UPDATE🟢******* 00.00.00
```python
SOOOON
```

`dymd version --long | head`
- version: v0.2.0-beta
- commit: 987e33407911c0578251f3ace95d2382be7e661d

```python
dymd init STAVRguide --chain-id=35-C
dymd config chain-id 35-C
```    

## Create/recover wallet
```python
dymd keys add <walletname>
dymd keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/obajay/nodes-Guides/main/Dymension/genesis.json -O $HOME/.dymension/config/genesis.json
```
`sha256sum $HOME/.dymension/config/genesis.json`
+ cf20e3b15d089ceeaaa9bb2abcd48a50f98e9f2274f4320aeae534d6972c4ee2

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0udym\"/;" ~/.dymension/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.dymension/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.dymension/config/config.toml
peers="b8d08951d68da03af8f9272bf77684811197c289@95.216.41.160:26656,5d689e09a129c03c003f05850262f03b2433a384@51.79.30.141:26656,8f84d324a2d266e612d06db4a793b0d001ee62a0@38.146.3.200:20556,43426e98064694d407b2165fb24d52980d38f1c9@88.99.3.158:20556,ee2fa87279bc626f9c979093389bd1d6568d96ff@65.109.37.228:36656,af6787b3273dd60e0f809c7e5e2a2a9fd379045e@195.201.195.61:27656,94b63fddfc78230f51aeb7ac34b9fb86bd042a77@146.19.24.43:30585,2d05753b4f5ac3bcd824afd96ea268d9c32ed84d@65.108.132.239:56656,d995d7079d975dea118a16014758838fe5cb8e2d@80.240.29.76:26656,f9d5e36ecc66b48f9fb940a778dd0c3b6b7c3d1d@65.109.106.211:26656,0d30a0790a216d01c9759ab48192d9154381e6c0@136.243.88.91:3240,cb1cc6b4c48b3e311f18b606c663c2dc0fb89b75@74.96.207.62:26656,5c2a752c9b1952dbed075c56c600c3a79b58c395@195.3.220.136:27086,e8a706e3a81a36a6dded6cc02eabaf5d355f4c1d@80.79.5.171:28656,7fc44e2651006fb2ddb4a56132e738da2845715f@65.108.6.45:61256,c6cdcc7f8e1a33f864956a8201c304741411f219@3.214.163.125:26656,55f233c7c4bea21a47d266921ca5fce657f3adf7@168.119.240.200:26656,db0264c412618949ce3a63cb07328d027e433372@146.19.24.101:26646"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml
seeds="c6cdcc7f8e1a33f864956a8201c304741411f219@3.214.163.125:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.dymension/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.dymension/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.dymension/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.dymension/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.dymension/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Dymension/addrbook.json"
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
# StateSync Dymension Testnet
```python
SNAP_RPC=http://dymension.rpc.t.stavr.tech:17087
peers="281190aa44ca82fb47afe60ba1a8902bae469b2a@dymension.peers.stavr.tech:17086"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.dymension/config/config.toml
dymd tendermint unsafe-reset-all --home /root/.dymension
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Dymension/addrbook.json"
systemctl restart dymd && journalctl -u dymd -f -o cat

```
# SnapShot Testnet (~0.9 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop dymd
cp $HOME/.dymension/data/priv_validator_state.json $HOME/.dymension/priv_validator_state.json.backup
rm -rf $HOME/.dymension/data
curl -o - -L http://dymension.snapshot.stavr.tech:1019/dymension/dymension-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.dymension --strip-components 2
mv $HOME/.dymension/priv_validator_state.json.backup $HOME/.dymension/data/priv_validator_state.json
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Dymension/addrbook.json"
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
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1000000" \
--amount 1000000udym \
--pubkey $(dymd tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id="35-C" \
--identity="" \
--website="" \
--details="" \
```

## Delete node
```python
sudo systemctl stop dymd && \
sudo systemctl disable dymd && \
rm /etc/systemd/system/dymd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf dymension && \
rm -rf .dymension && \
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
