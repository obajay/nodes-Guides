# Arkeo Testnet guide

![arkeo](https://github.com/obajay/nodes-Guides/assets/44331529/68f8a9ef-ae6b-4903-acab-547a08b712b4)

[WebSite](https://arkeo.network/)\
[GitHub](https://github.com/arkeonetwork/arkeo#arkeo-binary)
=
[EXPLORER 1](https://explorer.stavr.tech/arkeo-testnet) \
[EXPLORER 2](https://exp.utsa.tech/arkeo-test)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
SOON
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

# Build 06.09.23
```python
cd $HOME
git clone https://github.com/arkeonetwork/arkeo && cd arkeo
wget https://share101.utsa.tech/arkeo/arkeod
chmod +x arkeod
mv arkeod $HOME/go/bin/

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`arkeod version --long | grep -e commit -e version`
- version: 1
- commit: 68c59e9057e306dd99cdf55ebf4e6b1876835dc8

```python
arkeod init STAVRguide --chain-id arkeo
arkeod config chain-id arkeo
```    

## Create/recover wallet
```python
arkeod keys add <walletname>
  OR
arkeod keys add <walletname> --recover
```

## Download Genesis
```python
curl -s http://seed.arkeo.network:26657/genesis | jq '.result.genesis' > $HOME/.arkeo/config/genesis.json

```
`sha256sum $HOME/.arkeo/config/genesis.json`
+ 214828d2dac5eaaa4d2e70dde63bd460dcc86ab9e5dd7868dbfa8c3186b6abf9

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uarkeo\"/;" ~/.arkeo/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.arkeo/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.arkeo/config/config.toml
seeds="20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:22856"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.arkeo/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.arkeo/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.arkeo/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.arkeo/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.arkeo/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.arkeo/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.arkeo/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.arkeo/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.arkeo/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Arkeo/addrbook.json"
```

# Create a service file
```python
tee /etc/systemd/system/arkeod.service > /dev/null <<EOF
[Unit]
Description=arkeod
After=network-online.target

[Service]
User=$USER
ExecStart=$(which arkeod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Arkeo Testnet
```python
SOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable arkeod
sudo systemctl restart arkeod && sudo journalctl -u arkeod -f -o cat
```

### Create validator
```python
arkeod tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 0.2 \
--min-self-delegation "1" \
--amount "1000000"uarkeo \
--pubkey $(arkeod tendermint show-validator) \
--moniker "STAVRGuide" \
--from STAVR1 \
--chain-id arkeo \
--gas 350000 \
--details="" \
--identity="" \
--website="" -y

```

## Delete node
```python
sudo systemctl stop arkeod
sudo systemctl disable arkeod
rm /etc/systemd/system/arkeod.service
sudo systemctl daemon-reload
cd $HOME
rm -rf arkeo
rm -rf .arkeo && \
rm -rf $(which arkeod)
```
#
### Sync Info
```python
arkeod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
arkeod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u arkeod -f -o cat
```
### Check Balance
```python
arkeod query bank balances tarkeo...addressjkl1yjgn7z09ua9vms259j
```
