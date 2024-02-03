# Althea Testnet guide

![Althea_Logo-BLUE_SIGNAL](https://user-images.githubusercontent.com/44331529/218240936-c2095305-1a28-45f6-8ccd-d068a4fe5754.svg)

[WebSite](https://www.althea.net/)\
[GitHub](https://github.com/althea-net)
=
[EXPLORER](https://explorer.stavr.tech/Althea-testnetL1/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O althe https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Althea/althe && chmod +x althe && ./althe
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
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

# Build 05.07.23
```python
cd $HOME && mkdir -p go/bin/
wget https://github.com/althea-net/althea-L1/releases/download/v0.5.5/althea-linux-amd64
chmod +x althea-linux-amd64
sudo mv althea-linux-amd64 $HOME/go/bin/althea

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`althea version --long | head`
- version: v0.5.5
- commit: 3ec8309dfd84e090af7703edc21a70c95641975c

```python
althea init STAVR_guide --chain-id althea_417834-3
althea config chain-id althea_417834-3
```    

## Create/recover wallet
```python
althea keys add <walletname>
  OR
althea keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.althea/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Althea/genesis.json"

```
`sha256sum $HOME/.althea/config/genesis.json`
+ 66401e3c2d3f2679f82bec6a53c9b9bc38737c1ecb700c0a85f0188f8840ddeb

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0aalthea\"/;" ~/.althea/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.althea/config/config.toml
peers="bc47f3e8f9134a812462e793d8767ef7334c0119@chainripper-2.althea.net:23296"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.althea/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.althea/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.althea/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.althea/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
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
wget -O $HOME/.althea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Althea/addrbook.json"
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
SNAP_RPC=https://althea.rpc.t.stavr.tech:443
peers="a1ef55814e2b9aa6c75fbdda52a0ce3d10aebfec@althea.peers.stavr.tech:17886"
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
wget -O $HOME/.althea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Althea/addrbook.json"
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
--amount 1000000000000000000aalthea \
--pubkey $(althea tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id althea_417834-3 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Althea)
=


## Delete node
```python
sudo systemctl stop althea
sudo systemctl disable althea
rm /etc/systemd/system/althea.service
sudo systemctl daemon-reload
cd $HOME
rm -rf althea-chain
rm -rf .althea
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
