<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 51556.364 | 0.00053464 | 27.564094961 |

<!-- END_TABLE -->















































[🔥OUR VALIDATOR🔥](https://restake.app/lumnetwork/lumvaloper1rs6733nm675q3hzx85dl75t3q893tfukyz22he)
=

# Lum Mainnet guide
![lum](https://github.com/obajay/nodes-Guides/assets/44331529/13f47aa0-4c25-4d2e-b016-63900a303acc)

[WebSite](https://lum.network/)\
[GitHub](https://github.com/lum-network)
=
[EXPLORER](https://explorer.stavr.tech/LumNetwork-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O lumm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Lum/lumm && chmod +x lumm && ./lumm
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

# Build 01.02.24
```python
cd $HOME
git clone https://github.com/lum-network/chain.git lum
cd lum
git checkout v1.6.4
make install

```
*******🟢UPDATE🟢******* 01.02.24
```python
cd $HOME/chain
git pull
git checkout v1.6.4
make install
lumd version --long | grep -e commit -e version
#version: 1.6.4
#commit: 808aad2ee95e4e43ed117de1e2c85acb6e0d9e10
sudo systemctl restart lumd && sudo journalctl -u lumd -f -o cat
```

`lumd version --long | head`
- version: 1.6.4
- commit: 808aad2ee95e4e43ed117de1e2c85acb6e0d9e10

```python
lumd init STAVR_guide --chain-id lum-network-1
lumd config chain-id lum-network-1
```    

## Create/recover wallet
```python
lumd keys add <walletname>
            OR
lumd keys add <walletname> --recover
```

## Download Genesis
```python
curl https://anode.team/Lum/main/genesis.json > ~/.lumd/config/genesis.json
```
`sha256sum $HOME/.lumd/config/genesis.json`
+ f42e1d49ca30f69ace60f5eb61416e9393d318083849e83d1fc33df4085462c0

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0001ulum\"/;" ~/.lumd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.lumd/config/config.toml
peers="b47626b9d78ed7ed3c413304387026f907c70cbe@peer-0.mainnet.lum.network:26656,19ad16527c98b782ee35df56b65a3a251bd99971@peer-1.mainnet.lum.network:26656,5ea36d78ae774c9086c2d3fc8b91f12aa4bf3029@46.101.251.76:26656,a7f8832cb8842f9fb118122354fff22d3051fb83@3.36.179.104:26656,9afac13ba62fbfaf8d06867c30007162511093c0@54.214.134.223:26656,433c60a5bc0a693484b7af26208922b84773117e@34.209.132.0:26656,8fafab32895a31a0d7f17de58eddb492c6ced6d1@185.194.219.83:36656,c06eae3d9ea779710bca44e03f57e961b59d63f1@82.65.223.126:46656,4166de0e7721b6eec9c776abf2c38c40e7f820c5@202.61.239.130:26656,5a29947212a2615e43dac54deb55356a162e173a@35.181.76.160:26656,2cda4d97de0449878da10e456b176dd0720fbcec@62.171.129.174:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lumd/config/config.toml
seeds="19ad16527c98b782ee35df56b65a3a251bd99971@peer-1.mainnet.lum.network:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.lumd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.lumd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.lumd/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.lumd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.lumd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.lumd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.lumd/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.lumd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.lumd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Lum/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/lumd.service > /dev/null <<EOF
[Unit]
Description=lumd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which lumd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Lum Mainnet
```python
SNAP_RPC=https://lum.rpc.m.stavr.tech:443
SEEDS=42d79514ca40e942004e94f90557644cf36e986a@lum.seed.stavr.tech:31316
cp $HOME/.lumd/data/priv_validator_state.json $HOME/.lumd/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.lumd/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.lumd/config/config.toml
lumd tendermint unsafe-reset-all --home $HOME/.lumd --keep-addr-book
mv $HOME/.lumd/priv_validator_state.json.backup $HOME/.lumd/data/priv_validator_state.json
sudo systemctl restart lumd && journalctl -u lumd -f -o cat
```
# SnapShot Mainnet (~0.5GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop lumd
cp $HOME/.lumd/data/priv_validator_state.json $HOME/.lumd/priv_validator_state.json.backup
rm -rf $HOME/.lumd/data
curl -o - -L https://lum.snapshot.stavr.tech/lumd/lumd-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.lumd --strip-components 2
mv $HOME/.lumd/priv_validator_state.json.backup $HOME/.lumd/data/priv_validator_state.json
wget -O $HOME/.lumd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Lum/addrbook.json"
sudo systemctl restart lumd && journalctl -u lumd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable lumd
sudo systemctl restart lumd && sudo journalctl -u lumd -f -o cat
```

### Create validator
```python
lumd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000ulum \
--pubkey $(lumd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id lum-network-1 \
--fees="20ulum" \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Lum)
=

## Delete node
```python
sudo systemctl stop lumd
sudo systemctl disable lumd
rm /etc/systemd/system/lumd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf lum
rm -rf .lumd
rm -rf $(which lumd)
```
#
### Sync Info
```python
lumd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
lumd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u lumd -f -o cat
```
### Check Balance
```python
lumd query bank balances lumd...addressjkl1yjgn7z09ua9vms259j
```

