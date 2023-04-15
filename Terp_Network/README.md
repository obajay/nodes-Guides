# Terp Mainnet guide

![terp](https://user-images.githubusercontent.com/44331529/232221589-beeb5e24-82b8-4eaf-ad51-9e3c7fecd79b.png)

[WebSite](https://terp.network/)\
[GitHub](https://github.com/terpnetwork/terp-core.git)
=
[EXPLORER](https://explorer.stavr.tech/terp-mainnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
SOOON
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

# Build 11.04.23
```python
cd $HOME
git clone https://github.com/terpnetwork/terp-core.git
cd terp-core
git checkout v1.0.0
make install
```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`terpd version --long`
- version: 1.0.0
- commit: 22f9b2992a9a113bff7b923f7f39c46ae0f61857

```python
terpd init STAVRguide --chain-id morocco-1
terpd config chain-id morocco-1
```    

## Create/recover wallet
```python
terpd keys add <walletname>
  OR
terpd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.althea/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Althea/genesis.json"

```
`sha256sum $HOME/.althea/config/genesis.json`
+ af9260b536bc83875ae335d43a1b467967616a439ac736b3d18d6167a404f0b9

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ualthea\"/;" ~/.althea/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.althea/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.althea/config/config.toml
peers="733e9d5f995c2866df9f2e1254551940f060a70c@51.159.159.112:26656,11e8f38e3c5601e4ab2333d5a5bbb108a39b8e1c@159.69.110.238:26656,a81cf8f7f330e2e09bec93c866214f7b3b336849@65.109.87.88:26356,83147260a704b75283ca6da218516ee0eaa82956@170.64.156.36:26656,617433cdf5411fc9241d0f77239f751a14669368@146.190.156.221:26656,856ac01afa0163c27b69e1b25464427310120924@85.25.134.23:26656,d320b861277a338daefec6e620daafe07fc5ee19@65.108.199.36:20036,8203297aacaea1d889fcf36240484c9efc217bbd@116.202.156.106:26656,c6e1ed7117cd56036cc51835945d155e9c474c01@167.235.144.3:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.althea/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.althea/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.althea/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.althea/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.althea/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.althea/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.althea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Althea/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/althea.service > /dev/null <<EOF
[Unit]
Description=althea
After=network-online.target

[Service]
User=$USER
ExecStart=$(which althea) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Althea Testnet
```python
SNAP_RPC=http://althea.rpc.t.stavr.tech:17887
peers="90d692d481c1c4739ba8a7045b5552fa8d410901@althea.peers.stavr.tech:17886"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.althea/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.althea/config/config.toml
althea tendermint unsafe-reset-all --home /root/.althea
systemctl restart althea && journalctl -u althea -f -o cat
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop althea
cp $HOME/.althea/data/priv_validator_state.json $HOME/.althea/priv_validator_state.json.backup
rm -rf $HOME/.althea/data
curl -o - -L http://althea.snapshot.stavr.tech:1020/althea/althea-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.althea --strip-components 2
mv $HOME/.althea/priv_validator_state.json.backup $HOME/.althea/data/priv_validator_state.json
wget -O $HOME/.althea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Althea/addrbook.json"
sudo systemctl restart althea && journalctl -u althea -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable althea
sudo systemctl restart althea && sudo journalctl -u althea -f -o cat
```

### Create validator
```python
althea tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000000000000000ualthea \
--pubkey $(althea tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id althea_7357-1 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop althea && \
sudo systemctl disable althea && \
rm /etc/systemd/system/althea.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf althea-chain && \
rm -rf .althea && \
rm -rf $(which althea)
```
#
### Sync Info
```python
althea status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
althea status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u althea -f -o cat
```
### Check Balance
```python
althea query bank balances althea...addressjkl1yjgn7z09ua9vms259j
```
