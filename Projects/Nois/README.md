<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 818692.211 | null | 0 |

<!-- END_TABLE -->

















































[🔥OUR VALIDATOR🔥](https://restake.app/nois/noisvaloper1enwhnv85g4n99a2kzg8gey22xu6u43l4cxj824)
=

# Nois MAINNET
![nnnoi](https://user-images.githubusercontent.com/44331529/191945004-1227fef0-a215-44f1-bcab-854acd66de00.png)

[Website](https://nois.network/)
=
[EXPLORER 1](http://explorer.stavr.tech/Nois-Mainnet/staking) \
[EXPLORER 2](https://exp.utsa.tech/nois/staking) \
[EXPLORER 3](https://nois.explorers.guru)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```Python
wget -O noism https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nois/noism && chmod +x noism && ./noism
```

# 2) Manual installation

### Preparing the server

```Python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19 

```Python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 11.01.24
```Python
cd $HOME
git clone https://github.com/noislabs/noisd
cd noisd
git checkout v1.0.5
make install
```
`noisd version --long | grep -e commit -e version`
- version: 1.0.5
- commit: 1e7b65f785b43e9b389ff7be058d935677fdaf78

```Python
noisd init STAVRguide --chain-id nois-1
noisd config chain-id nois-1
```    

## Create/recover wallet
```Python
noisd keys add <walletname>
    OR
noisd keys add <walletname> --recover
```

## Download Genesis

```Python
wget -O $HOME/.noisd/config/genesis.json https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nois/genesis.json
```
`sha256sum $HOME/.noisd/config/genesis.json`
+ 5332fb6477a2d273fd7e5a13bceb213e2a9d408a698c49ab34e8b78736e58cac

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```Python
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.noisd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.noisd/config/config.toml
peers="b3e3bd436ee34c39055a4c9946a02feec232988c@seeds.cros-nest.com:56656,babc3f3f7804933265ec9c40ad94f4da8e9e0017@seed.rhinostake.com:17356,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:17356,72cd4222818d25da5206092c3efc2c0dd0ec34fe@161.97.96.91:36656,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:17356"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.noisd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.noisd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.noisd/config/config.toml
export CONFIG_DIR="$HOME/.noisd/config"
# Update app.toml
sed -i 's/minimum-gas-prices =.*$/minimum-gas-prices = "0.05unois"/' $CONFIG_DIR/app.toml

```
### Pruning (optional)
```Python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.noisd/config/app.toml
```
### Indexer (optional) 
```Python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.noisd/config/config.toml
```

## Download addrbook
```Python
wget -O $HOME/.noisd/config/addrbook.json https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nois/addrbook.json
```

# StateSync Mainnet
```Python
SNAP_RPC=https://nois.rpc.m.stavr.tech:443
peers="9fa9b59890187293a8f6b57d1f606fdfe751396e@nois.peer.stavr.tech:40136"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.noisd/config/config.toml
noisd tendermint unsafe-reset-all --home $HOME/.noisd
wget -O $HOME/.noisd/config/addrbook.json https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nois/addrbook.json
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.noisd/config/app.toml
curl -o - -L http://nois.wasm.stavr.tech:1004/wasm-nois.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.noisd --strip-components 2
systemctl restart noisd && journalctl -u noisd -f -o cat

```

# SnapShot Mainnet updated every 5 hours 
```Python
cd $HOME
apt install lz4
sudo systemctl stop noisd
cp $HOME/.noisd/data/priv_validator_state.json $HOME/.noisd/priv_validator_state.json.backup
rm -rf $HOME/.noisd/data
curl -o - -L http://nois.snapshot.stavr.tech:1028/noisd/noisd-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.noisd --strip-components 2
curl -o - -L http://nois.wasm.stavr.tech:1004/wasm-nois.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.noisd --strip-components 2
mv $HOME/.noisd/priv_validator_state.json.backup $HOME/.noisd/data/priv_validator_state.json
wget -O $HOME/.noisd/config/addrbook.json https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nois/addrbook.json
sudo systemctl restart noisd && journalctl -u noisd -f -o cat
```


# Create a service file
```Python
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=noisd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noisd) start --home $HOME/.noisd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```Python
sudo systemctl daemon-reload
sudo systemctl enable noisd
sudo systemctl restart noisd && sudo journalctl -u noisd -f -o cat
```

### Create validator
```Python
noisd tx staking create-validator \
--amount=1000000unois \
--pubkey=$(noisd tendermint show-validator) \
--moniker=STAVRguide \
--chain-id=nois-1 \
--commission-rate="0.05" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--from=<wallet> \
--fees=5000unois \
--gas=300000 \
-y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Nois)
=

## Delete node
```Python
sudo systemctl stop noisd && \
sudo systemctl disable noisd && \
rm /etc/systemd/system/noisd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf noisd && \
rm -rf .noisd && \
rm -rf $(which noisd)
```
