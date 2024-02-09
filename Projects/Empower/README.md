[🔥OUR VALIDATOR🔥](https://restake.app/empowerchain/empowervaloper1c4jqjpjxm55hhxvssyuahd3zs3h567x0jyzmx0)
=

# Empower MAINNET Guide
![empower](https://user-images.githubusercontent.com/44331529/193969092-38e7ec7f-bca0-4bd9-a31d-5ba52b71ec81.png)

[Website](https://www.empowerchain.io/)
=
[EXPLORER](http://explorer.stavr.tech/Empower-Mainnet/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 160GB    |

# 1) Auto_install script
```python
wget -O empwm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Empower/empwm && chmod +x empwm && ./empwm
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.20 (one command)
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


# Binary 20.12.23
```python
cd $HOME
wget https://github.com/EmpowerPlastic/empowerchain/releases/download/v2.0.0/empowerd-v2.0.0-linux-amd64.zip
unzip empowerd-v2.0.0-linux-amd64.zip && rm -rf empowerd-v2.0.0-linux-amd64.zip
chmod +x empowerd
mv empowerd $HOME/go/bin/
```
*******🟢UPDATE🟢******* 20.12.23
```python
cd $HOME && wget https://github.com/EmpowerPlastic/empowerchain/releases/download/v2.0.0/empowerd-v2.0.0-linux-amd64.zip
unzip empowerd-v2.0.0-linux-amd64.zip
rm -rf empowerd-v2.0.0-linux-amd64.zip
chmod +x empowerd
mv empowerd $(which empowerd)
empowerd version --long | grep -e version -e commit
#commit: 70ad47fc878d1854fe279ebf99e3a9260b78099c
#version: v2.0.0
systemctl restart empowerd && journalctl -u empowerd -f -o cat
```

`empowerd version --long`
- version: v2.0.0
- commit: 70ad47fc878d1854fe279ebf99e3a9260b78099c


## Initialisation
```python
empowerd init STAVRguide --chain-id empowerchain-1
empowerd config chain-id empowerchain-1

```
## Add wallet
```python
empowerd keys add <walletName>
empowerd keys add <walletName> --recover
```
# Genesis
```python
URL=https://github.com/EmpowerPlastic/empowerchain/raw/main/mainnet/empowerchain-1/genesis.tar.gz
curl -L $URL | tar -xz -C $HOME/.empowerchain/config/
```

`sha256sum $HOME/.empowerchain/config/genesis.json`
- 819d33d14c35bbfbc5997db9bf545eb7a5504b5870a307ce90c3813add4b316b

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.empowerchain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.empowerchain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.empowerchain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.empowerchain/config/app.toml
```

### Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.empowerchain/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025umpwr\"/" $HOME/.empowerchain/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.empowerchain/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.empowerchain/config/config.toml
seeds="a1427b456513ab70967a2a5c618d347bc89e8848@seed.empowerchain.io:26656,6740fa259552a628266a85de8c2a3dee7702b8f9@empower-mainnet-seed.itrocket.net:14656,e16668ddd526f4e114ebb6c4714f0c18c0add8f8@empower-seed.zenscape.one:26656,f2ed98cf518b501b6d1c10c4a16d0dfbc4a9cc98@tenderseed.ccvalidators.com:27001"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.empowerchain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.empowerchain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.empowerchain/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.empowerchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Empower/addrbook.json"
```
# StateSync
```python
SNAP_RPC=https://empw.rpc.m.stavr.tech:443
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.empowerchain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.empowerchain/config/config.toml
empowerd tendermint unsafe-reset-all --home $HOME/.empowerchain
curl -o - -L https://empw.wasm.stavr.tech/wasm-empw.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.empowerchain --strip-components 2
wget -O $HOME/.empowerchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Empower/addrbook.json"
systemctl restart empowerd && journalctl -u empowerd -f -o cat
```
# SnapShot (~0.5 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop empowerd
cp $HOME/.empowerchain/data/priv_validator_state.json $HOME/.empowerchain/priv_validator_state.json.backup
rm -rf $HOME/.empowerchain/data
curl -o - -L https://empw-m.snapshot.stavr.tech/empw-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.empowerchain --strip-components 2
curl -o - -L https://empw.wasm.stavr.tech/wasm-empw.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.empowerchain --strip-components 2
mv $HOME/.empowerchain/priv_validator_state.json.backup $HOME/.empowerchain/data/priv_validator_state.json
wget -O $HOME/.empowerchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Empower/addrbook.json"
sudo systemctl restart empowerd && sudo journalctl -u empowerd -f -o cat
```

# Create a service file
```python
sudo tee /etc/systemd/system/empowerd.service > /dev/null <<EOF
[Unit]
Description=EmpowerChain Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which empowerd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

# Start node (one command)
```python
sudo systemctl daemon-reload && sudo systemctl enable empowerd
sudo systemctl restart empowerd && sudo journalctl -u empowerd -f -o cat
```

## Create validator
```python
empowerd tx staking create-validator \
--amount 1000000umpwr \
--from <walletname> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(empowerd tendermint show-validator) \
--moniker STAVRguide \
--chain-id empowerchain-1 \
-y
```

### Delete node (one command)
```python
sudo systemctl stop empowerd && \
sudo systemctl disable empowerd && \
rm /etc/systemd/system/empowerd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .empowerchain && \
rm -rf empowerchain && \
rm -rf $(which empowerd)
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Empower)
=
