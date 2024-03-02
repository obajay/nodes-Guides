<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 537869.9 | 0.02304766 | 12396.643084984 |

<!-- END_TABLE -->

































































































[🔥OUR VALIDATOR🔥](https://restake.app/vidulum/vdlvaloper1ad4n75v6wu4n39frrryfya9mw77h4qz4j6gv6r)
=

# Vidulum Mainnet guide
![vidulum](https://github.com/obajay/nodes-Guides/assets/44331529/b9a11d68-57da-47db-88ab-95f21593ca54)


[WebSite](https://vidulum.app/)\
[GitHub](https://github.com/vidulum)
=
[EXPLORER](https://explorer.stavr.tech/Vidulum-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O vdlm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Vidulum/vdlm && chmod +x vdlm && ./vdlm
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

# Build 18.10.22
```python
cd $HOME
git clone https://github.com/vidulum/mainnet vidulum
cd vidulum
git checkout v1.2.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`vidulumd version --long | head`
- version: 1.2.0
- commit: latest

```python
vidulumd init STAVRguide --chain-id vidulum-1
vidulumd config chain-id vidulum-1
```    

## Create/recover wallet
```python
vidulumd keys add <walletname>
            OR
vidulumd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.vidulum/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Vidulum/genesis.json"

```
`sha256sum $HOME/.vidulum/config/genesis.json`
+ fa4c12c5c7e796a3804960cd502edb8a3171f3c5dd421a1ce69b96351cd785a3

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001uvdl\"/;" ~/.vidulum/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.vidulum/config/config.toml
peers="b63833b8b8740660ae3ac87f058447465f91f8f4@65.109.28.177:26726"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.vidulum/config/config.toml
seeds="883ec7d5af7222c206674c20c997ccc5c242b38b@ec2-3-82-120-39.compute-1.amazonaws.com:26656,eed11fff15b1eca8016c6a0194d86e4a60a65f9b@apollo.erialos.me:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.vidulum/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.vidulum/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.vidulum/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.vidulum/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.vidulum/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.vidulum/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.vidulum/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.vidulum/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.vidulum/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Vidulum/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/vidulumd.service > /dev/null <<EOF
[Unit]
Description=vidulumd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which vidulumd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Vidulum Mainnet
```python
SNAP_RPC=https://vidulum.rpc.m.stavr.tech:443
SEEDS=197f4d559555de6b7fe360c6a926ca8812a749be@vidulum.peer.stavr.tech:1046
cp $HOME/.vidulum/data/priv_validator_state.json $HOME/.vidulum/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.vidulum/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.vidulum/config/config.toml
vidulumd tendermint unsafe-reset-all --home $HOME/.vidulum --keep-addr-book
mv $HOME/.vidulum/priv_validator_state.json.backup $HOME/.vidulum/data/priv_validator_state.json
sudo systemctl restart vidulumd && journalctl -u vidulumd -f -o cat
```
# SnapShot Mainnet (~0.2 GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop vidulumd
cp $HOME/.vidulum/data/priv_validator_state.json $HOME/.vidulum/priv_validator_state.json.backup
rm -rf $HOME/.vidulum/data
curl -o - -L https://vidulum.snapshot.stavr.tech/vidulum-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.vidulum --strip-components 2
mv $HOME/.vidulum/priv_validator_state.json.backup $HOME/.vidulum/data/priv_validator_state.json
wget -O $HOME/.vidulum/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Vidulum/addrbook.json"
sudo systemctl restart vidulumd && journalctl -u vidulumd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable vidulumd
sudo systemctl restart vidulumd && sudo journalctl -u vidulumd -f -o cat
```

### Create validator
```python
vidulumd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000uvdl \
--pubkey $(vidulumd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id vidulum-1 \
--fees="20000uvdl" \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Vidulum)
=

## Delete node
```python
sudo systemctl stop vidulumd
sudo systemctl disable vidulumd
rm /etc/systemd/system/vidulumd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf vidulum
rm -rf .vidulum
rm -rf $(which vidulumd)
```
#
### Sync Info
```python
vidulumd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
vidulumd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u vidulumd -f -o cat
```
### Check Balance
```python
vidulumd query bank balances vdl...addressjkl1yjgn7z09ua9vms259j
```
