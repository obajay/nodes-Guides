# Medibloc Mainnet guide

![mdeib](https://github.com/obajay/nodes-Guides/assets/44331529/c633e31a-5533-4ce0-b5c8-4d43fdbfd655)

[WebSite](https://medibloc.com/) \
[GitHub](https://github.com/medibloc/panacea-mainnet/tree/master)
=
[EXPLORER 1](https://explorer.stavr.tech/Medibloc-Mainnet/) \
[EXPLORER 2](https://www.mintscan.io/medibloc/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O mediblocm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Medibloc/mediblocm && chmod +x mediblocm && ./mediblocm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.20.5
```python
ver="1.20.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 04.07.23
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/medibloc/panacea-core
cd panacea-core
git checkout v2.0.7-2
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`panacead version --long | grep -e commit -e version`
- version: 2.0.7-2
- commit: 7202206201b78cb3c72ca05c557f0f8af3dda593

```python
panacead init STAVR_guide --chain-id panacea-3
panacead config chain-id panacea-3
```    

## Create/recover wallet
```python
panacead keys add <walletname>
  OR
panacead keys add <walletname> --recover
```

## Download Genesis
```python
wget https://github.com/medibloc/panacea-mainnet/raw/master/panacea-3/genesis.json.gz
gzip -d genesis.json.gz && mv genesis.json ~/.panacea/config/
```
`sha256sum $HOME/.panacea/config/genesis.json`
+ e8fa05ce0bbb021a7fc5bf0abd8b6c1840c983ef17d4a8d09ba2646b4abfee33

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.1umed\"/;" ~/.panacea/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.panacea/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.panacea/config/config.toml
peers="395aead00e99f828e4af92531dcd8c8da1255a8f@3.36.50.133:26656,c238f279c970764d6893ae44bdf5c949dc22b009@13.114.44.199:26656,00c57e36559b49ce7d29fa4920b5132584994368@52.77.227.241:26656,5cd589ab0f34dbeb07cb0e156741838b2c7d3737@148.251.235.130:16656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.panacea/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.panacea/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.panacea/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.panacea/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.panacea/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.panacea/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.panacea/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.panacea/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.panacea/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.panacea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Medibloc/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/panacead.service > /dev/null <<EOF
[Unit]
Description=panacead
After=network-online.target

[Service]
User=$USER
ExecStart=$(which panacead) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Medibloc Mainnet
```python
SNAP_RPC=https://panacea.rpc.m.stavr.tech:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.panacea/config/config.toml
panacead tendermint unsafe-reset-all
wget -O $HOME/.panacea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Medibloc/addrbook.json"
sudo systemctl restart panacead && sudo journalctl -u panacead -f -o cat
```
# SnapShot Mainnet (~0.9GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop panacead
cp $HOME/.panacea/data/priv_validator_state.json $HOME/.panacea/priv_validator_state.json.backup
rm -rf $HOME/.panacea/data
curl -o - -L https://panacea.snapshot.stavr.tech/panacea/panacea-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.panacea --strip-components 2
mv $HOME/.panacea/priv_validator_state.json.backup $HOME/.panacea/data/priv_validator_state.json
wget -O $HOME/.panacea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Medibloc/addrbook.json"
sudo systemctl restart panacead && journalctl -u panacead -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable panacead
sudo systemctl restart panacead && sudo journalctl -u panacead -f -o cat
```

### Create validator
```python
panacead tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000udsm \
--pubkey $(panacead tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id panacea-3 \
--gas 300000 \
--fees 30000umed \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Medibloc)
=


## Delete node
```python
sudo systemctl stop panacead
sudo systemctl disable panacead
rm /etc/systemd/system/panacead.service
sudo systemctl daemon-reload
cd $HOME
rm -rf panacea-core
rm -rf .panacea
rm -rf $(which panacead)
```
#
### Sync Info
```python
panacead status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
panacead status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u panacead -f -o cat
```
### Check Balance
```python
panacead query bank balances panacea...addressjkl1yjgn7z09ua9vms259j
```
