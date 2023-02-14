# Dymension Testnet guide

![dymension](https://user-images.githubusercontent.com/44331529/216242184-e602001a-8794-495a-81fc-b0d10589963e.png)


[WebSite](https://dymension.xyz/) \
[GitHub](https://github.com/dymensionxyz/testnets)
=
[EXPLORER](https://explorer.stavr.tech/dymension-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O dym https://raw.githubusercontent.com/obajay/nodes-Guides/main/Dymension/dym && chmod +x dym && ./dym
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
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

# Build 01.02.23
```python
cd $HOME
git clone https://github.com/dymensionxyz/dymension.git --branch v0.2.0-beta
cd dymension
make install

```
*******🟢UPDATE🟢******* 00.00.00
```python
SOOOON
```

`dymd version --long | head`
- version: v0.2.0-beta
- commit: 987e33407911c0578251f3ace95d2382be7e661d

```python
dymd init STAVRguide --chain-id=35-C
dymd config chain-id 35-C
```    

## Create/recover wallet
```python
dymd keys add <walletname>
dymd keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/obajay/nodes-Guides/main/Dymension/genesis.json -O $HOME/.dymension/config/genesis.json
```
`sha256sum $HOME/.dymension/config/genesis.json`
+ cf20e3b15d089ceeaaa9bb2abcd48a50f98e9f2274f4320aeae534d6972c4ee2

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0udym\"/;" ~/.dymension/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.dymension/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.dymension/config/config.toml
peers="dc237ba44f4f178f6a72b60d9dee2337d424bfce@65.109.85.226:26656,3515bc6054d3e71caf2e04effaad8c95ee4b6dc6@165.232.186.173:26656,e9a375501c0a2eab296a16753667c708ed64649e@95.214.53.46:26656,2d05753b4f5ac3bcd824afd96ea268d9c32ed84d@65.108.132.239:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml
seeds="c6cdcc7f8e1a33f864956a8201c304741411f219@3.214.163.125:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.dymension/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.dymension/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.dymension/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.dymension/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.dymension/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.dymension/config/addrbook.json "SOOOOOOOOON
```

# Create a service file
```python
sudo tee /etc/systemd/system/dymd.service > /dev/null <<EOF
[Unit]
Description=dymd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which dymd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Dymension Testnet
```python
SOOOOOOOOOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable dymd
sudo systemctl restart dymd && sudo journalctl -u dymd -f -o cat
```

### Create validator
```python
dymd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1000000" \
--amount 1000000udym \
--pubkey $(dymd tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id="35-C" \
--identity="" \
--website="" \
--details="" \
```

## Delete node
```python
sudo systemctl stop dymd && \
sudo systemctl disable dymd && \
rm /etc/systemd/system/dymd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf dymension && \
rm -rf .dymension && \
rm -rf $(which dymd)
```
#
### Sync Info
```python
dymd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
dymd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u dymd -f -o cat
```
### Check Balance
```python
dymd query bank balances dym...addressjkl1yjgn7z09ua9vms259j
```
