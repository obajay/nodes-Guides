<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 1440008.037 | 0.03567191 | 51367.837112298 |

<!-- END_TABLE -->



































































































[🔥OUR VALIDATOR🔥](https://restake.app/source/sourcevaloper13l78szv3mxcru9mhh2ndv58xla2arfzhv045cx)
=

# Guide Source MAINNET
![sour (2)](https://user-images.githubusercontent.com/44331529/183239082-09722b8d-9cc7-49a1-9d93-15ce3ab8d752.png)
![sour (1)](https://user-images.githubusercontent.com/44331529/183239083-d3ac3a34-0cc4-4e8b-aafc-42a5e7a3f7f5.png)

[Website](https://www.sourceprotocol.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/Source-Mainnet/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 260GB    |

# 1) Auto_install script 
```python
wget -O sourcem https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Source/sourcem && chmod +x sourcem && ./sourcem
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
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

# Binary   17.01.24
```python 
cd $HOME
git clone https://github.com/Source-Protocol-Cosmos/source.git
cd ~/source
git checkout v3.0.1
make install
```
`sourced version --long | head`
- version: v3.0.1
- commit: 2aae45f6e2d23f6c81c02075b74ee9f5b7598e3e

## Initialisation
```python
sourced init STAVR_guide --chain-id=source-1
sourced config chain-id source-1
```
## Add wallet
```python
sourced keys add <walletName>
    or
sourced keys add <walletName> --recover
```
# Genesis
```python
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/mainnet/master/source-1/genesis.json > ~/.source/config/genesis.json
```

`sha256sum $HOME/.source/config/genesis.json`
- ba2261082818227073bd8b49717a9781bf5c440c8e34e21ec72fb15806f047cc  genesis.json

### Pruning (optional) one command
```python
pruning="custom" && \
pruning_keep_recent="1000" && \
pruning_keep_every="0" && \
pruning_interval="100" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.source/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.source/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.source/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.source/config/app.toml
```
### Indexer (optional) one command
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.source/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.25usource\"/;" ~/.source/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.source/config/config.toml
peers="96d63849a529a15f037a28c276ea6e3ac2449695@34.222.1.252:26656,0107ac60e43f3b3d395fea706cb54877a3241d21@35.87.85.162:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.source/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.source/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.source/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.source/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.source/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Source/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/sourced.service > /dev/null <<EOF
[Unit]
Description=source
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sourced) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## StateSync Mainnet
```python
SNAP_RPC=https://source.rpc.m.stavr.tech:443
peers="9751bfbbb3303db1898ef5c601d8522938623262@source.peers.stavr.tech:20056"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.source/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.source/config/config.toml
sourced tendermint unsafe-reset-all
sudo systemctl restart sourced && journalctl -u sourced -f -o cat
```

# SnapShot (~0.2 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop sourced
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" ~/.source/config/config.toml
cp $HOME/.source/data/priv_validator_state.json $HOME/.source/priv_validator_state.json.backup
rm -rf $HOME/.source/data
curl -o - -L https://source-m.snapshot.stavr.tech/source/source-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.source --strip-components 2
wget -O $HOME/.source/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Source/addrbook.json"
mv $HOME/.source/priv_validator_state.json.backup $HOME/.source/data/priv_validator_state.json
sudo systemctl restart sourced && journalctl -u sourced -f -o cat
```

# Start node (one command)
```python
sudo systemctl daemon-reload &&
sudo systemctl enable sourced &&
sudo systemctl restart sourced && sudo journalctl -u sourced -f -o cat
```

## Create validator
```python
sourced tx staking create-validator \
--amount=1000000usource \
--pubkey=$(sourced tendermint show-validator) \
--moniker=STAVR_Guide \
--chain-id=source-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees=100usource \
--from=<walletName> \
--identity="" \
--website="" \
--fees 50000usource \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Source)
=

### Delete node (one command)
```python
sudo systemctl stop sourced
sudo systemctl disable sourced
rm /etc/systemd/system/sourced.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .source
rm -rf source
rm -rf $(which sourced)
```

