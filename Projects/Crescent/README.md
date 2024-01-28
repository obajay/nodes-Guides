# Crescent Mainnet guide

![cres](https://github.com/obajay/nodes-Guides/assets/44331529/13cc0725-6963-4bf9-8cc7-40b9d30a6b3f)

[WebSite](https://crescent.network/) \
[GitHub](https://github.com/crescent-network/crescent)
=
[EXPLORER 1](https://explorer.stavr.tech/Crescent-Mainnet) \
[EXPLORER 2](https://ping.pub/crescent/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O crescentm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crescent/crescentm && chmod +x crescentm && ./crescentm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
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

# Build 11.01.23
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/crescent-network/crescent
cd crescent
git checkout v4.2.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`crescentd version --long | grep -e commit -e version`
- version: 4.2.0
- commit: c816c3730a6bd3ed2c370846b0943bddf2f45f2a

```python
crescentd init STAVR_guide --chain-id crescent-1
crescentd config chain-id crescent-1
```    

## Create/recover wallet
```python
crescentd keys add <walletname>
  OR
crescentd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O genesis.json https://snapshots.polkachu.com/genesis/crescent/genesis.json --inet4-only
mv genesis.json ~/.crescent/config
```
`sha256sum $HOME/.crescent/config/genesis.json`
+ 88da94b7346eed173dbdab425272c692ec0f37627c7b8b43ffd3fd0397264273

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ucre\"/;" ~/.crescent/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.crescent/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.crescent/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.crescent/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:14556"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.crescent/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.crescent/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.crescent/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.crescent/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.crescent/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.crescent/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.crescent/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.crescent/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.crescent/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crescent/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/crescentd.service > /dev/null <<EOF
[Unit]
Description=crescentd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which crescentd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Crescent Mainnet
```python
cd $HOME
SNAP_RPC=https://crescent.rpc.m.stavr.tech:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.crescent/config/config.toml
crescentd tendermint unsafe-reset-all
wget -O $HOME/.crescent/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crescent/addrbook.json"
sudo systemctl restart crescentd && sudo journalctl -u crescentd -f -o cat
```
# SnapShot Mainnet (~0.9GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop crescentd
cp $HOME/.crescent/data/priv_validator_state.json $HOME/.crescent/priv_validator_state.json.backup
rm -rf $HOME/.crescent/data
curl -o - -L https://crescent.snapshot.stavr.tech/crescent/crescent-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.crescent --strip-components 2
mv $HOME/.crescent/priv_validator_state.json.backup $HOME/.crescent/data/priv_validator_state.json
wget -O $HOME/.crescent/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crescent/addrbook.json"
sudo systemctl restart crescentd && journalctl -u crescentd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable crescentd
sudo systemctl restart crescentd && sudo journalctl -u crescentd -f -o cat
```

### Create validator
```python
crescentd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 100000000ucre \
--pubkey $(crescentd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id crescent-1 \
--fees 3000ucre \
--gas 300000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Crescent)
=


## Delete node
```python
sudo systemctl stop crescentd
sudo systemctl disable crescentd
rm /etc/systemd/system/crescentd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf crescent
rm -rf .crescent
rm -rf $(which crescentd)
```
#
### Sync Info
```python
crescentd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
crescentd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u crescentd -f -o cat
```
### Check Balance
```python
crescentd query bank balances cre...addressjkl1yjgn7z09ua9vms259j
```
