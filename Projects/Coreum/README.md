# Coreum Testnet guide

![coreum (2)](https://user-images.githubusercontent.com/44331529/218271606-8fb90d09-f25e-446d-bda6-de87595eda68.png)

[WebSite](https://www.coreum.com/) \
[GitHub](https://github.com/CoreumFoundation)
=
[EXPLORER 1](https://explorer.stavr.tech/Coreum-testnet/staking)\
[EXPLORER 2](https://explorer.nodexcapital.com/coreum/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O core https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Coreum/core && chmod +x core && ./core
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

# Build 11.02.23
```python
cd $HOME
curl -LOf https://github.com/CoreumFoundation/coreum/releases/download/v0.1.1/cored-linux-amd64
chmod +x cored-linux-amd64
mv cored-linux-amd64 $HOME/go/bin/cored
```
`cored version --long`
- version: v0.1.1
- commit: 4ed5ee97e49e4fc004854b3c9486348c9f373cfd

```python
cored config chain-id coreum-testnet-1
cored init STAVRguide --chain-id coreum-testnet-1
```    

## Create/recover wallet
```python
cored keys add <walletname>
          or 
cored keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.core/coreum-testnet-1/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Coreum/genesis.json"
```
`sha256sum $HOME/.core/coreum-testnet-1/config/genesis.json`
+ 8ece5edef851738ef3d8435a58bfbfe91b163ff199785ae1470da84466f0f1c1

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
SEEDS="64391878009b8804d90fda13805e45041f492155@35.232.157.206:26656,53f2367d8f8291af8e3b6ca60efded0675ff6314@34.29.15.170:26656"
PEERS="69d7028b7b3c40f64ea43208ecdd43e88c797fd6@34.69.126.231:26656,b2978432c0126f28a6be7d62892f8ded1e48d227@34.70.241.13:26656,7c0d4ce5ad561c3453e2e837d85c9745b76f7972@35.238.77.191:26656,0aa5fa2507ada8a555d156920c0b09f0d633b0f9@34.173.227.148:26656,4b8d541efbb343effa1b5079de0b17d2566ac0fd@34.172.70.24:26656,27450dc5adcebc84ccd831b42fcd73cb69970881@35.239.146.40:26656,5add70ec357311d07d10a730b4ec25107399e83c@5.196.7.58:26656,1a3a573c53a4b90ab04eb47d160f4d3d6aa58000@35.233.117.165:26656,abbeb588ad88176a8d7592cd8706ebbf7ef20cfe@185.241.151.197:26656,39a34cd4f1e908a88a726b2444c6a407f67e4229@158.160.59.199:26656,051a07f1018cfdd6c24bebb3094179a6ceda2482@138.201.123.234:26656,cc6d4220633104885b89e2e0545e04b8162d69b5@75.119.134.20:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.core/coreum-testnet-1/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utestcore\"/;" ~/.core/coreum-testnet-1/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.core/coreum-testnet-1/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.core/coreum-testnet-1/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.core/coreum-testnet-1/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.core/coreum-testnet-1/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.core/coreum-testnet-1/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.core/coreum-testnet-1/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.core/coreum-testnet-1/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.core/coreum-testnet-1/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.core/coreum-testnet-1/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.core/coreum-testnet-1/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.core/coreum-testnet-1/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Coreum/addrbook.json"
```
# StateSync Coreum Testnet
```python
SNAP_RPC=http://coreum.rpc.t.stavr.tech:52657
peers=eb5c90c1e845b8a2a989b20fd528097569956ebf@coreum.peer.stavr.tech:17686
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.core/coreum-testnet-1/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.core/coreum-testnet-1/config/config.toml
cored tendermint unsafe-reset-all
systemctl restart cored && journalctl -u cored -f -o cat
```
# SnapShot (~0.2 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop cored
cp $HOME/.core/coreum-testnet-1/data/priv_validator_state.json $HOME/.core/coreum-testnet-1/priv_validator_state.json.backup
rm -rf $HOME/.core/coreum-testnet-1/data
curl -o - -L http://coreum.snapshot.stavr.tech:1022/cored/cored-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.core/coreum-testnet-1 --strip-components 3
mv $HOME/.core/coreum-testnet-1/priv_validator_state.json.backup $HOME/.core/coreum-testnet-1/data/priv_validator_state.json
wget -O $HOME/.core/coreum-testnet-1/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Coreum/addrbook.json"
sudo systemctl restart cored && journalctl -u cored -f -o cat
```

# Create a service file
```python
sudo tee /etc/systemd/system/cored.service > /dev/null <<EOF
[Unit]
Description=cored
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cored) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload && sudo systemctl enable cored
sudo systemctl restart cored && sudo journalctl -u cored -f -o cat
```

### Create validator
```python
cored tx staking create-validator \
  --amount 2000000000utestcore \
  --from wallet \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "2000000000" \
  --pubkey  $(cored tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id coreum-testnet-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```python
sudo systemctl stop cored && \
sudo systemctl disable cored && \
rm /etc/systemd/system/cored.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .core && \
rm -rf $(which cored)
```
#
### Sync Info
```python
source $HOME/.bash_profile
cored status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
cored status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u cored -f -o cat
```
### Check Balance
```python
cored query bank balances testcore...address1yjgn7z09ua9vms259j
```
