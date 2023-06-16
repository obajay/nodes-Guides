# Cosmic Mainnet guide

![cosmic](https://github.com/obajay/nodes-Guides/assets/44331529/95976b09-b2ee-41ca-9b9d-a1a607af3888)


[WebSite](https://cosmic-horizon.com/)\
[GitHub](https://github.com/cosmic-horizon/QWOYN)
=
[EXPLORER](https://explorer.stavr.tech/cosmic-mainnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   8| 16GB | 150GB    |


# 1) Auto_install script
```python
SOOON
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

# Build 16.06.23
```python
cd $HOME
git clone https://github.com/althea-net/althea-chain
cd QWOYN
git checkout v5.0.2
make install
```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`qwoynd version --long`
- version: 5.0.2
- commit: b1608d57401d4d498e39737446206ce062385b4f

```python
qwoynd init STAVRguide --chain-id qwoyn-1
qwoynd config chain-id qwoyn-1
```    

## Create/recover wallet
```python
qwoynd keys add <walletname>
  OR
qwoynd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.qwoynd/config/genesis.json "https://raw.githubusercontent.com/cosmic-horizon/mainnet/main/genesis.json"
```
`sha256sum $HOME/.qwoynd/config/genesis.json`
+ 9dd858eb43729f8b53a919212eec25df6a34d02b9c45079d3341923b1f3e67bd

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025uqwoyn\"/;" ~/.qwoynd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.qwoynd/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.qwoynd/config/config.toml
seeds="3ba8858372b8b6314b29d43f8fc344cc54f4ffe0@45.77.126.244:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.qwoynd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.qwoynd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.qwoynd/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.qwoynd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.qwoynd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.qwoynd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.qwoynd/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.qwoynd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.qwoynd/config/addrbook.json "SOOOOOOOON"
```

# Create a service file
```python
sudo tee /etc/systemd/system/qwoynd.service > /dev/null <<EOF
[Unit]
Description=qwoynd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which qwoynd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Cosmic Testnet
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
sudo systemctl enable qwoynd
sudo systemctl restart qwoynd && sudo journalctl -u qwoynd -f -o cat
```

### Create validator
```python
qwoynd tx staking create-validator \
  --amount=1000000uqwoyn \
  --pubkey=$(qwoynd tendermint show-validator) \
  --moniker="STAVRguide" \
  --chain-id="qwoyn-1" \
  --commission-rate="0.01" \
  --commission-max-rate="0.25" \
  --commission-max-change-rate="0.2" \
  --details="" \
  --identity="" \
  --website="" \
  --min-self-delegation="1" \
  --gas="400000" \
  --from=wallet \
  --fees="1000uqwoyn" -y
```

## Delete node
```python
sudo systemctl stop qwoynd
sudo systemctl disable qwoynd
rm /etc/systemd/system/qwoynd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf QWOYN
rm -rf .qwoynd
rm -rf $(which qwoynd)
```
#
### Sync Info
```python
qwoynd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
qwoynd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u qwoynd -f -o cat
```
### Check Balance
```python
qwoynd query bank balances qwoynd...addressjkl1yjgn7z09ua9vms259j
```
