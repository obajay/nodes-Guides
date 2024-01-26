<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 1135.460 | 0.00339734 | 3.857543826 |

<!-- END_TABLE -->

























[🔥OUR VALIDATOR🔥](https://restake.app/likecoin/likevaloper1pxvegy2m8u3v4c20p5p4v0wre4ggd5luumku9w)
=

# Likecoin Mainnet guide
![liked](https://github.com/obajay/nodes-Guides/assets/44331529/93c6794e-ca4d-4c89-832b-c89fdca918f2)

[WebSite](https://about.like.co/)\
[GitHub](https://github.com/likecoin)
=
[EXPLORER](https://explorer.stavr.tech/Likecoin-M)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O likem https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Likecoin/likem && chmod +x likem && ./likem
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
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

# Build 07.12.23
```python
cd $HOME
git clone https://github.com/likecoin/likecoin-chain
cd likecoin-chain
git checkout v4.1.1
make install

```
*******🟢UPDATE🟢******* 07.12.23
```python
cd $HOME/likecoin-chain
git pull
git checkout v4.1.1
make install
liked version --long | grep -e commit -e version
#version: v4.1.1
#commit: e84c626f03c92cc19a35a6e9fd57f29d113dbb1d
sudo systemctl restart lumd && sudo journalctl -u lumd -f -o cat
```

`liked version --long | head`
- version: 4.1.1
- commit: e84c626f03c92cc19a35a6e9fd57f29d113dbb1d

```python
liked init STAVRguide --chain-id likecoin-mainnet-2
liked config chain-id likecoin-mainnet-2
```    

## Create/recover wallet
```python
liked keys add <walletname>
            OR
liked keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.liked/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Likecoin/genesis.json"

```
`sha256sum $HOME/.liked/config/genesis.json`
+ f42e1d49ca30f69ace60f5eb61416e9393d318083849e83d1fc33df4085462c0

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"10000nanolike\"/;" ~/.liked/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.liked/config/config.toml
peers="d8eaf867e1ec1d1c3bc872a93bf0f060701d10be@65.109.28.177:29696"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.liked/config/config.toml
seeds="7a38dfc59eb43b27cf2cc87b46a43e76aeaaf012@20.205.224.107:26656,49976c3bd43da9271f226cbedf02d4b6b8fc880c@35.233.143.230:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.liked/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.liked/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.liked/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.liked/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.liked/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.liked/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.liked/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.liked/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.liked/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Likecoin/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/liked.service > /dev/null <<EOF
[Unit]
Description=liked
After=network-online.target

[Service]
User=$USER
ExecStart=$(which liked) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Likecoin Mainnet
```python
SNAP_RPC=https://like.rpc.m.stavr.tech:443
SEEDS=fd7589625f4ad41bb93f96f4c962ed6638426497@like.peer.stavr.tech:1006
cp $HOME/.liked/data/priv_validator_state.json $HOME/.liked/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.liked/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.liked/config/config.toml
liked tendermint unsafe-reset-all --home $HOME/.liked --keep-addr-book
mv $HOME/.liked/priv_validator_state.json.backup $HOME/.liked/data/priv_validator_state.json
wget -O $HOME/.liked/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Likecoin/addrbook.json"
sudo systemctl restart liked && journalctl -u liked -f -o cat
```
# SnapShot Mainnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable liked
sudo systemctl restart liked && sudo journalctl -u liked -f -o cat
```

### Create validator
```python
liked tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000nanolike \
--pubkey $(liked tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id likecoin-mainnet-2 \
--fees="200000nanolike" \
--identity="" \
--website="" \
--details="" -y
```
[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Likecoin)
=

## Delete node
```python
sudo systemctl stop liked
sudo systemctl disable liked
rm /etc/systemd/system/liked.service
sudo systemctl daemon-reload
cd $HOME
rm -rf likecoin-chain
rm -rf .liked
rm -rf $(which liked)
```
#
### Sync Info
```python
liked status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
liked status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u liked -f -o cat
```
### Check Balance
```python
liked query bank balances like...addressjkl1yjgn7z09ua9vms259j
```
