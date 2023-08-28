<h1 align="center"> 🔥C4E TESTNET guide🔥</h1>

![c4e (2)](https://user-images.githubusercontent.com/44331529/216780015-d723d176-ae86-403e-aab6-7d7e6254a144.png)


[WebSite](https://c4e.io/) \
[GitHub](https://github.com/chain4energy/c4e-chain.git)
=
[EXPLORER 1](https://explorer.stavr.tech/c4e-testnet/staking) \
[EXPLORER 2](https://explorer-testnet.c4e.io/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   8| 16GB | 250GB   |


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

## GO 1.20.3
```python
cd $HOME
ver="1.20.3"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 29.08.23
```python
cd $HOME
git clone https://github.com/chain4energy/c4e-chain.git
cd c4e-chain
git checkout tags/v1.3.0
```
*******🟢UPDATE🟢******* 29.08.23
```python
cd $HOME/c4e-chain
git fetch --all
git checkout v1.3.0
make install
c4ed version --long
#commit: 272f5bd2f5c0fa54686f3eef2a2d64bf0ad4c50f
#version: 1.3.0
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```

`c4ed version --long`
- version: v1.3.0
- commit: 272f5bd2f5c0fa54686f3eef2a2d64bf0ad4c50f

```python
c4ed init STAVRguide --chain-id babajaga-1
c4ed config chain-id babajaga-1
```    

## Create/recover wallet
```python
c4ed keys add <walletname>
  OR
c4ed keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/chain4energy/c4e-chains/main/babajaga-1/genesis.json -O $HOME/.c4e-chain/config/genesis.json
```

`sha256sum $HOME/.c4e-chain/config/genesis.json`
+ 0b862736c62b1fba573b1623f4bc8eb2a12000549f538bb6b31c2d1d57f57956

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uc4e\"/" $HOME/.c4e-chain/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.c4e-chain/config/config.toml
peers="de18fc6b4a5a76bd30f65ebb28f880095b5dd58b@66.70.177.76:36656,33f90a0ac7e8f48305ea7e64610b789bbbb33224@151.80.19.186:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.c4e-chain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
```

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.c4e-chain/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.c4e-chain/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.c4e-chain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/C4E/C4E_Testnet/addrbook.json"
```
## StateSync (MZONDER)
```python
RPC="http://66.70.177.76:36657"
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.c4e-chain/config/config.toml
c4ed tendermint unsafe-reset-all --home /root/.c4e-chain
sudo systemctl restart c4ed && journalctl -u c4ed -f -o cat
```
## SnapShot (~0.1 GB) updated every 5 hours
```python
SOOON
```

# Create a service file
```python
sudo tee /etc/systemd/system/c4ed.service > /dev/null <<EOF
[Unit]
Description=c4e
After=network-online.target

[Service]
User=$USER
ExecStart=$(which c4ed) start
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
sudo systemctl enable c4ed
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```

### Create validator
```python
c4ed tx staking create-validator \
  --amount 1000000uc4e \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(c4ed tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id babajaga-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```python
sudo systemctl stop c4ed && \
sudo systemctl disable c4ed && \
rm /etc/systemd/system/c4ed.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf c4e-chain && \
rm -rf .c4e-chain && \
rm -rf $(which c4ed)
```
#
### Sync Info
```python
source $HOME/.bash_profile
c4ed status 2>&1 | jq .SyncInfo
```
### Node Info
```python
c4ed status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u c4ed -f -o cat
```
### Check Balance
```python
c4ed query bank balances c4e...address1yjgn7z09ua9vms259j
```
