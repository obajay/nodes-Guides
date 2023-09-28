# Guide Source
![sour (2)](https://user-images.githubusercontent.com/44331529/183239082-09722b8d-9cc7-49a1-9d93-15ce3ab8d752.png)
![sour (1)](https://user-images.githubusercontent.com/44331529/183239083-d3ac3a34-0cc4-4e8b-aafc-42a5e7a3f7f5.png)

[Website](https://www.sourceprotocol.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/Source/staking) \
[EXPLORER 2](https://testnet.itrocket.net/source/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4| 8GB  | 160GB    |

# 1) Auto_install script 
```python
wget -O sources https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Source/sources && chmod +x sources && ./sources
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

# Binary   28.09.23
```python 
cd $HOME
git clone https://github.com/Source-Protocol-Cosmos/source.git
cd ~/source
git checkout v3.0.0
make install
```
`sourced version --long | head`
- version: v3.0.0
- commit: 39030aa705cf18819b0a439c1bd646ed9195728c 

## Initialisation
```python
sourced init STAVRguide --chain-id=sourcetest-1
sourced config chain-id sourcetest-1
```
## Add wallet
```python
sourced keys add <walletName>
    or
sourced keys add <walletName> --recover
```
# Genesis
```python
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/testnets/master/sourcetest-1/genesis.json > ~/.source/config/genesis.json
```

`sha256sum $HOME/.source/config/genesis.json`
- c8b8e28f1cc2c6bb708d963146842da9e367874267d90ab99a13a6bd736d5682  genesis.json

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
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"1usource\"/;" ~/.source/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.source/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.source/config/config.toml
peers="ace839c852739d1ea6e3675d30380fe085c1c23a@52.26.226.21:26656,8145d4d13511e7f89dbd257f51ed5d076941f12f@164.92.98.12:26656"
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

# SnapShot (~0.1 GB) updated every 6 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop sourced
cp $HOME/.source/data/priv_validator_state.json $HOME/.source/priv_validator_state.json.backup
rm -rf $HOME/.source/data
curl -o - -L http://source.snapshot.stavr.tech:4001/source/source-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.source --strip-components 2
curl -o - -L http://source.wasm.stavr.tech:1050/wasm-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.source/data --strip-components 3
mv $HOME/.source/priv_validator_state.json.backup $HOME/.source/data/priv_validator_state.json
wget -O $HOME/.source/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Source/addrbook.json"
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" ~/.source/config/config.toml
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
--moniker=STAVR_guide \
--chain-id=sourcetest-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees=100usource \
--from=<walletName> \
--identity="" \
--website="" \
--fees 200000usource \
--details="" -y
```

### Delete node (one command)
```python
sudo systemctl stop sourced && \
sudo systemctl disable sourced && \
rm /etc/systemd/system/sourced.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .source && \
rm -rf source && \
rm -rf $(which sourced)
```

