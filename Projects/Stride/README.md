<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 25509.4 | 5.24 | 133669.731833920 |

<!-- END_TABLE -->
















[🔥OUR VALIDATOR🔥](https://restake.app/stride/stridevaloper1n94ndmxqf7vke553lr3ewwt4edtc4g6mdyx9qn)
=

# Guide Stride 
![stride](https://user-images.githubusercontent.com/44331529/180614293-57dff376-2d34-4480-803a-e8262bf37fdd.png)

[Website](https://stride.zone/)
=
[EXPLORER 1](https://explorer.stavr.tech/Stride/staking) \
[EXPLORER 2](https://www.mintscan.io/stride/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |

# 1) Auto_install script
```python
wget -O stride-x https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Stride/stride-x && chmod +x stride-x && ./stride-x
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
```

## GO 1.20.5 (one command) 
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
# Binary   30.01.24
```python
cd $HOME
git clone https://github.com/Stride-Labs/stride.git && cd stride
git checkout v18.0.0
make install
```
*******🟢UPDATE🟢******* 30.01.24
```python
cd $HOME/stride
git fetch --all
git checkout v18.0.0
make install
strided version --long | grep -e commit -e version
#commit: 4913e1dd1a9b0e6e55678d3eede25f7c8a5b43a2
#version: v18.0.0
sudo systemctl restart strided && journalctl -u strided -f -o cat
```

`strided version`
+ commit: 4913e1dd1a9b0e6e55678d3eede25f7c8a5b43a2
+ version: v18.0.0


## Initialisation
```python
strided init STAVRguide --chain-id stride-1
```
## Add wallet
```python
strided keys add <walletName>
strided keys add <walletName> --recover
```
# Genesis
```python
wget -O $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Stride/genesis.json"
```

`sha256sum $HOME/.stride/config/genesis.json`
- e31b8d1a9090f128b6f7db024836213d1af2b424bfb757e38f3b5c39be738aa4  genesis.json

### Pruning 
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stride/config/app.toml
```
### Indexer (optional)
```python
ndexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stride/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ustrd\"/;" ~/.stride/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.stride/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.stride/config/config.toml

peers="3a6ea526f5f0857272c79448850abef71653d4df@83.136.249.85:26656,fbc05b4136a15b0b69a2b7f093731453f04ce2f4@192.168.50.57:26656,6795c4cd27d6132a547888ed5b7996aee454a025@172.25.0.2:26656,34c52450d2f107b7c164eb103641df9e45a322d4@65.21.192.108:26656,681803f48d5e9ef9870918e8330551513eccb31c@78.47.51.53:26656,0f2d7f17589e6e31691649ec04fe19561c0d12a6@10.138.0.7:26656,cb0b38aa612e8ac05f704d9b2feb7526607afb77@159.203.191.62:26656,55b446443f2bd68e06200c3f294c735c333722b0@162.251.235.252:26656,68fb634620e00a5a18f606360b6ca6d989da8ce6@65.108.106.131:26656,f56ddd6af02efaac4c47cc8053685d11c1065996@0.0.0.0:26656,f1721dd0324f29c108c1072b2e4fe9c64e63122d@192.168.86.25:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stride/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.stride/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 10/g' $HOME/.stride/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 40/g' $HOME/.stride/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Stride/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/strided.service > /dev/null <<EOF
[Unit]
Description=strided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which strided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync STRIDE (Temporarily suspended)
```python
SNAP_RPC=http://stride.rpc.m.stavr.tech:21017
peers="a7b4cf6f65138ba61518c2c45402da32dc8e28b7@stride.peer.stavr.tech:21016"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.stride/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 300)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.stride/config/config.toml
strided tendermint unsafe-reset-all --home /root/.stride --keep-addr-book
sudo systemctl restart strided && journalctl -u strided -f -o cat
```
# SnapShot (~0.8GB) updated every 5 hours  (Temporarily suspended)
```python
cd $HOME
apt install lz4
sudo systemctl stop strided
cp $HOME/.stride/data/priv_validator_state.json $HOME/.stride/priv_validator_state.json.backup
rm -rf $HOME/.stride/data
curl -o - -L http://stride.snapshot.stavr.tech:1008/stride/stride-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.stride --strip-components 2
mv $HOME/.stride/priv_validator_state.json.backup $HOME/.stride/data/priv_validator_state.json
wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Stride/addrbook.json"
sudo systemctl restart strided && journalctl -u strided -f -o cat
```

# Start node (one command)
```python
sudo systemctl daemon-reload &&
sudo systemctl enable strided &&
sudo systemctl restart strided && sudo journalctl -u strided -f -o cat
```

## Create validator
    strided tx staking create-validator \
    --amount=1000000ustrd \
    --pubkey=$(strided tendermint show-validator) \
    --moniker=STAVRguide \
    --chain-id=stride-1 \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.1" \
    --min-self-delegation="1" \
    --fees=500ustrd \
    --from=<walletName> \
    --identity="" \
    --website="" \
    --details="" \
    -y


### Delete node (one command)
```python
sudo systemctl stop strided
sudo systemctl disable strided
rm /etc/systemd/system/strided.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .stride
rm -rf stride
rm -rf $(which strided)
```
