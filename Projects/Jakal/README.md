<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 367475.9 | 0.780787 | 286920.465245 |

<!-- END_TABLE -->











[🔥OUR VALIDATOR🔥](https://restake.app/jackal/jklvaloper1us3q2ytkn9zyn99gvf66u6nsn3wnq0n3kxpyvm)
=

# Jaсkal Mainnet guide
![jackal](https://user-images.githubusercontent.com/44331529/198365498-60a1a986-70e8-419d-a35c-b4d9780c0e7a.png)

[GitHub](https://github.com/JackalLabs)
=
[EXPLORER 1](https://explorer.stavr.tech/Jackal/staking) \
[EXPLORER 2](https://explorer.nodestake.top/jackal/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O jkl https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Jakal/jkl && chmod +x jkl && ./jkl
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.21.4

```python
ver="1.21.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 22.02.24
```python
cd $HOME
git clone https://github.com/JackalLabs/canine-chain && cd canine-chain
git checkout v3.1.3
make install
```

*******🟢UPDATE🟢******* 22.02.24
```python
cd $HOME/canine-chain
git fetch --all
git checkout v3.1.3
make install
canined version --long | head
#version: 3.1.3
#commit: 4d3ddc3f943c5e78debc1e7c9c84555bc273df63
systemctl restart canined && journalctl -u canined -f -o cat
```

`canined version --long | head`
- 4d3ddc3f943c5e78debc1e7c9c84555bc273df63
- 3.1.3

```python
canined init STAVRguide --chain-id jackal-1

```    

## Create/recover wallet
```python
canined keys add <walletname>
canined keys add <walletname> --recover
```

## Download Genesis
```python
wget https://jackal-m.genesis.stavr.tech/genesis.json
mv genesis.json $HOME/.canine/config/genesis.json
```
`sha256sum $HOME/.canine/config/genesis.json`
+ b6a0e67fbdf21b929d9080ac546db080790bc18c4931bea60fec8fb18559ad39

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ujkl\"/;" ~/.canine/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.canine/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.canine/config/config.toml
peers="c98028a169aabe5f8555090e6cd3370308b391b3@135.181.20.44:2506,ec38fb158ffb0272c4b7c951fc790a8f9849e280@198.244.212.27:26656,ff94a29e02de8369faf37c76d3c97684bbd51bd6@185.16.38.165:17556,39b55b1c49ad0994bbead006be40d9c84b0bf2d4@78.107.253.133:28656,f90a64a0a3f3c0480360e0fe5dd0f806d7741558@207.244.127.5:26656,57d82676ab660e8e4471664d7fee18e3e2e3dd19@89.58.38.59:26656,8be44995ab4eeafcde6e0a9e196c40d483ef6d2a@51.81.155.97:10556"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.canine/config/config.toml
seeds="ec38fb158ffb0272c4b7c951fc790a8f9849e280@198.244.212.27:26656"
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
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.canine/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.canine/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Jakal/addrbook.json"
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

# StateSync Jackal
```python
SNAP_RPC=https://jkl.rpc.m.stavr.tech:443
peers="ddb821309deba8f274b18ef3ae8731f239569b5c@jkl.rpc.m.stavr.tech:11126"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.canine/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 300)); \
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
# SnapShot (~1.3GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop canined
cp $HOME/.canine/data/priv_validator_state.json $HOME/.canine/priv_validator_state.json.backup
rm -rf $HOME/.canine/data
curl -o - -L http://jkl.snapshot.stavr.tech:1018/jackal/jackal-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.canine --strip-components 2
mv $HOME/.canine/priv_validator_state.json.backup $HOME/.canine/data/priv_validator_state.json
wget -O $HOME/.canine/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Jakal/addrbook.json"
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
  --amount 1000000ujkl \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(canined tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id jackal-1 \
  --identity="" \
  --details="" \
  --website="" -y
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
