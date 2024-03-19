<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 520418.3 | 0.01432556 | 7455.284293986 |

<!-- END_TABLE -->



















































[🔥OUR VALIDATOR🔥](https://restake.app/teritori/torivaloper1sqk72uwf6tg867ssuu7whxfu9pfcyrpeqwa92c)
=

# Teritori Mainnet guide
![tert](https://user-images.githubusercontent.com/44331529/180614436-1041172a-0b1e-4df3-85b7-3d18899f3e43.png)

[EXPLORER](http://explorer.stavr.tech/Teritori-Main/staking)
=
- **Recommended hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  8 | 16GB | 200GB    |

# 1) Auto_install script 
```python
wget -O teritorm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Teritori/teritorm && chmod +x teritorm && ./teritorm
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 19 (one command)
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

## Build 29.01.24
```python
git clone https://github.com/TERITORI/teritori-chain
cd teritori-chain 
git checkout v2.0.6
make install
```

*******🟢UPDATE🟢******* 29.01.24
```python
cd $HOME/teritori-chain
git fetch --all
git checkout v2.0.6
make install
sudo systemctl restart teritorid && sudo journalctl -u teritorid -f -o cat
```
`teritorid version --long`
+ version: 2.0.6
+ commit: 99fa787ee24ccf82c005da8e369e00afd938874a

```python
teritorid init STAVR_guide --chain-id teritori-1
teritorid config chain-id teritori-1
```

## Create/recover wallet
```python
teritorid keys add <walletname>
teritorid keys add <walletname> --recover
```

### when creating, do not forget to write down the seed phrase

# Genesis
```python
cd $HOME
wget -O ~/.teritorid/config/genesis.json https://media.githubusercontent.com/media/TERITORI/teritori-chain/v1.1.2/mainnet/teritori-1/genesis.json

```

## Seeds,peers and gas price
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0utori\"/;" ~/.teritorid/config/app.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.teritorid/config/config.toml

peers="3069b058b5ed85c3cdb2cf18fb1d255d966b53af@193.149.187.8:26656,a06fbbb9ace823ae28a696a91daa2d0644653c28@65.21.32.200:26756,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:15956"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.teritorid/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.teritorid/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Teritori/addrbook.json"
```

## Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.teritorid/config/app.toml
```

## Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.teritorid/config/config.toml
```

# StateSync
```python
SNAP_RPC=https://teritori.rpc.m.stavr.tech:443
peers="ab77fecd8c58d89a1bd28bc198449aa2d7fb8740@teritori.peers.stavr.tech:38026"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.teritorid/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 300)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.teritorid/config/config.toml
teritorid tendermint unsafe-reset-all --home $HOME/.teritorid --keep-addr-book
curl -o - -L http://teritori.wasm.stavr.tech:1011/wasm-teritori.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.teritorid --strip-components 2
wget -O $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Teritori/addrbook.json"
sudo systemctl restart teritorid && journalctl -u teritorid -f -o cat
```
# SnapShot (~0.7 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop teritorid
cp $HOME/.teritorid/data/priv_validator_state.json $HOME/.teritorid/priv_validator_state.json.backup
rm -rf $HOME/.teritorid/data
curl -o - -L http://teritori.snapshot.stavr.tech:1001/teritori/teritori-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.teritorid --strip-components 2
mv $HOME/.teritorid/priv_validator_state.json.backup $HOME/.teritorid/data/priv_validator_state.json
wget -O $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Teritori/addrbook.json"
sudo systemctl restart teritorid && journalctl -u teritorid -f -o cat
```

# Create a service file
```python
sudo tee /etc/systemd/system/teritorid.service > /dev/null <<EOF
[Unit]
Description=Teritorid
After=network-online.target

[Service]
User=$USER
ExecStart=$(which teritorid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

    
# Start
```python
sudo systemctl daemon-reload
sudo systemctl enable teritorid
sudo systemctl restart teritorid
sudo journalctl -u teritorid -f -o cat
```

## Create validator
```python
teritorid tx staking create-validator \
--amount=1000000utori \
--pubkey=$(teritorid tendermint show-validator) \
--moniker="STAVR_guide" \
--identity="" \
--details="" \
--website="" \
--chain-id="teritori-1" \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees 500utori \
--from=<walletname> -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Teritori)
=

# Delete node 
```python
sudo systemctl stop teritorid
sudo systemctl disable teritorid
rm /etc/systemd/system/teritorid.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .teritorid
rm -rf teritori-chain
rm -rf $(which teritorid)
```
