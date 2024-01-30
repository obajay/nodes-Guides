# Jaсkal Testnet guide

![jackal](https://user-images.githubusercontent.com/44331529/198365498-60a1a986-70e8-419d-a35c-b4d9780c0e7a.png)


[GitHub](https://github.com/JackalLabs/jackal-chain-assets/tree/main/testnet)
=
[EXPLORER 1](https://explorer.stavr.tech/Jackal-Testnet/staking) \
[EXPLORER 2](https://testnet-explorer.brocha.in/jackal/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O jkltest https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Jakal/Jackal-Testnet/jkltest && chmod +x jkltest && ./jkltest
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.20
```python
ver="1.20"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 30.01.24
```python
cd $HOME
git clone https://github.com/JackalLabs/canine-chain
cd canine-chain
git checkout v3.1.2-rc.1
make install
```
*******🟢UPDATE🟢******* 30.01.24
```python
cd $HOME/canine-chain
git fetch --all
git checkout v3.1.2-rc.1
make install
canined version --long | head
#version: 3.1.2
#commit: 9a88bf707fb3c114edb8f69f6f3f0a2487aba0e6
sudo systemctl restart canined && sudo journalctl -u canined -f -o cat
```

`canined version --long | head`
- version: 3.1.2
- commit: 9a88bf707fb3c114edb8f69f6f3f0a2487aba0e6

```python
canined init STAVR_guide --chain-id lupulella-2
canined config chain-id lupulella-2
```    

## Create/recover wallet
```python
canined keys add <walletname>
canined keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.canine/config/genesis.json "https://raw.githubusercontent.com/JackalLabs/jackal-chain-assets/main/testnet/genesis.json"
```
`sha256sum $HOME/.canine/config/genesis.json`
+ 9701001c2188abf7c117f7192030bcbab358ac1d5b1a61594f443d3b206ab5a2

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ujkl\"/;" ~/.canine/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.canine/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.canine/config/config.toml
peers="5eedbfbe64b942f4ab54db3842acf3bfab034c24@161.97.74.88:46656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.canine/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.canine/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.canine/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.canine/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.canine/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.canine/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.canine/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.canine/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.canine/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.canine/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Jakal/Jackal-Testnet/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/canined.service > /dev/null <<EOF
[Unit]
Description=canined
After=network-online.target

[Service]
User=$USER
ExecStart=$(which canined) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Jackal Testnet
```python
SNAP_RPC=http://jkl.rpc.t.stavr.tech:19127
peers="80613772b20df144945801b42f327d0945a24374@jkltest.peer.stavr.tech:19126"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.canine/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.canine/config/config.toml
canined tendermint unsafe-reset-all --home /root/.canine --keep-addr-book
systemctl restart canined && journalctl -u canined -f -o cat
```
# SnapShot Testnet (~0.5 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop canined
cp $HOME/.canine/data/priv_validator_state.json $HOME/.canine/priv_validator_state.json.backup
rm -rf $HOME/.canine/data
curl -o - -L http://jkltest.snapshot.stavr.tech:1015/jackalt/jackalt-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.canine --strip-components 2
mv $HOME/.canine/priv_validator_state.json.backup $HOME/.canine/data/priv_validator_state.json
wget -O $HOME/.canine/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Jakal/Jackal-Testnet/addrbook.json"
sudo systemctl restart canined && journalctl -u canined -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable canined
sudo systemctl restart canined && sudo journalctl -u canined -f -o cat
```

### Create validator
```python
canined tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1000000" \
--amount 1000000ujkl \
--pubkey $(canined tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--identity="" \
--website="" \
--details="" \
--fees 500ujkl
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Jackal)
=

## Delete node
```python
sudo systemctl stop canined && \
sudo systemctl disable canined && \
rm /etc/systemd/system/canined.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf canine-chain && \
rm -rf .canine && \
rm -rf $(which canined)
```
#
### Sync Info
```python
canined status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
canined status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u canined -f -o cat
```
### Check Balance
```python
canined query bank balances jkl...addressjkl1yjgn7z09ua9vms259j
```
