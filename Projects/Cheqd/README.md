<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 2982113.527 | 0.0954 | 284493.630543 |

<!-- END_TABLE -->




















[🔥OUR VALIDATOR🔥](https://restake.app/cheqd/cheqdvaloper167t2yzx24s0n92esd9xd6zwmy2ya92teac7evx)
=

# CHEQD Mainnet guide
![ceqd](https://github.com/obajay/nodes-Guides/assets/44331529/ddd2b6c7-cf35-4c26-bf0c-fd47be08c9bd)

[WebSite](https://cheqd.io/)\
[GitHub](https://github.com/cheqd)
=
[EXPLORER](https://explorer.stavr.tech/Cheqd-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O cheqdm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cheqd/cheqdm && chmod +x cheqdm && ./cheqdm
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

# Build 09.06.23
```python
cd $HOME
git clone https://github.com/cheqd/cheqd-node
cd cheqd-node
git checkout v1.4.4
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`cheqd-noded version --long`
- version: HEAD-4147fce724a775558940c63d5593fee3ea6e00ab
- commit: 4147fce724a775558940c63d5593fee3ea6e00ab

```python
cheqd-noded init STAVRguide --chain-id cheqd-mainnet-1
cheqd-noded config chain-id cheqd-mainnet-1
```    

## Create/recover wallet
```python
cheqd-noded keys add <walletname>
            OR
cheqd-noded keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.cheqdnode/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cheqd/genesis.json"

```
`sha256sum $HOME/.cheqdnode/config/genesis.json`
+ f42e1d49ca30f69ace60f5eb61416e9393d318083849e83d1fc33df4085462c0

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"25ncheq\"/;" ~/.cheqdnode/config/app.toml
external_address=$(wget -qO- eth0.me)
sed -i 's/log_level =.*/log_level = "info"/g' $HOME/.cheqdnode/config/config.toml
sed -i 's/log_format =.*/log_format = "plain"/g' $HOME/.cheqdnode/config/config.toml
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.cheqdnode/config/config.toml
peers="fc6f7914e4beb4b5278e7ba32ec2abde97cd8082@65.109.28.177:26336"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.cheqdnode/config/config.toml
seeds="258a9bfb822637bfca87daaab6181c10e7fd0910@seed1.eu.cheqd.net:26656,f565ff792b20977face9817df6acb268d41d4092@seed2.eu.cheqd.net:26656,388947cc7d901c5c06fedc4c26751634564d68e6@seed3.eu.cheqd.net:26656,9b30307a2a2819790d68c04bb62f5cf4028f447e@seed1.ap.cheqd.net:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.cheqdnode/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.cheqdnode/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.cheqdnode/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cheqdnode/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cheqdnode/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.cheqdnode/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cheqdnode/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.cheqdnode/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.cheqdnode/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cheqd/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/cheqd-noded.service > /dev/null <<EOF
[Unit]
Description=cheqd-noded
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cheqd-noded) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Cheqd Mainnet
```python
SNAP_RPC=https://cheqd.rpc.m.stavr.tech:443
SEEDS=46bb1e68fcc2750ecdc4253986d653f4bd7228ef@cheqd.peer.stavr.tech:21016
cp $HOME/.cheqdnode/data/priv_validator_state.json $HOME/.cheqdnode/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.cheqdnode/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.cheqdnode/config/config.toml
cheqd-noded tendermint unsafe-reset-all --home $HOME/.cheqdnode --keep-addr-book
mv $HOME/.cheqdnode/priv_validator_state.json.backup $HOME/.cheqdnode/data/priv_validator_state.json
sudo systemctl restart cheqd-noded && journalctl -u cheqd-noded -f -o cat
```
# SnapShot Mainnet (~3 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop cheqd-noded
cp $HOME/.cheqdnode/data/priv_validator_state.json $HOME/.cheqdnode/priv_validator_state.json.backup
rm -rf $HOME/.cheqdnode/data
curl -o - -L http://cheqd.snapshot.stavr.tech:4/cheqd/cheqd-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.cheqdnode --strip-components 2
mv $HOME/.cheqdnode/priv_validator_state.json.backup $HOME/.cheqdnode/data/priv_validator_state.json
wget -O $HOME/.cheqdnode/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cheqd/addrbook.json"
sudo systemctl restart cheqd-noded && journalctl -u cheqd-noded -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable cheqd-noded
sudo systemctl restart cheqd-noded && sudo journalctl -u cheqd-noded -f -o cat
```

### Create validator
```python
cheqd-noded tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000ncheq \
--pubkey $(cheqd-noded tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id cheqd-mainnet-1 \
--fees="5000000ncheq" \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Cheqd)
=


## Delete node
```python
sudo systemctl stop cheqd-noded
sudo systemctl disable cheqd-noded
rm /etc/systemd/system/cheqd-noded.service
sudo systemctl daemon-reload
cd $HOME
rm -rf cheqd-node
rm -rf .cheqdnode
rm -rf $(which cheqd-noded)
```
#
### Sync Info
```python
cheqd-noded status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
cheqd-noded status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u cheqd-noded -f -o cat
```
### Check Balance
```python
cheqd-noded query bank balances cheqd...addressjkl1yjgn7z09ua9vms259j
```
