# Cascadia Testnet guide

![cascadia](https://user-images.githubusercontent.com/44331529/232544057-cd6fdd6c-1fc4-4048-87db-030364cec75a.png)

[WebSite](https://www.cascadia.foundation/)\
[GitHub](https://github.com/cascadiafoundation/cascadia)
=
[EXPLORER](https://explorer.stavr.tech/Cascadia-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


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

## GO 1.20.3
```python
ver="1.20.3"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 09.10.23
```python
cd $HOME
git clone https://github.com/cascadiafoundation/cascadia && cd cascadia
git checkout v0.1.6
make install
```
*******🟢UPDATE🟢******* 09.10.23
```python
cd $HOME
wget https://github.com/CascadiaFoundation/cascadia/releases/download/v0.1.6/cascadiad
chmod +x cascadiad
mv cascadiad $(which cascadiad)
cascadiad version --long | grep -e version -e commit
#commit: 06ad7b0222b3d7795c25173231fb8b90ad79cdbf
#version: 0.1.6
sudo systemctl restart cascadiad && sudo journalctl -u cascadiad -f -o cat
```

`cascadiad version --long | grep -e version -e commit`
- version: v0.1.6
- commit: 06ad7b0222b3d7795c25173231fb8b90ad79cdbf

```python
cascadiad init STAVRguide --chain-id cascadia_6102-1
cascadiad config chain-id cascadia_6102-1
```    

## Create/recover wallet
```python
cascadiad keys add <walletname>
  OR
cascadiad keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.cascadiad/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cascadia/genesis.json"

```
`sha256sum $HOME/.cascadiad/config/genesis.json`
+ 74ea3c84182028300d0c101c5cf017a055782c595ed91e4be3638380f0169582

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025aCC\"/;" ~/.cascadiad/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.cascadiad/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.cascadiad/config/config.toml
peers="1d61222b7b8e180aacebfd57fbd2d8ab95ebdc4c@65.109.93.152:35656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.cascadiad/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.cascadiad/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.cascadiad/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.cascadiad/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.cascadiad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cascadiad/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.cascadiad/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.cascadiad/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cascadia/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/cascadiad.service > /dev/null <<EOF
[Unit]
Description=Cascadia Foundation
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cascadiad) start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Cascadian Testnet
```python
SOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable cascadiad
sudo systemctl restart cascadiad && sudo journalctl -u cascadiad -f -o cat
```

### Create validator
```python
cascadiad tx staking create-validator \
  --amount "10000000000000000000"aCC \
  --from wallet \
  --commission-max-change-rate "0.2" \
  --commission-max-rate "0.5" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(cascadiad tendermint show-validator) \
  --moniker "STAVRguide" \
  --chain-id cascadia_6102-1 \
  --details="" \
  --identity="" \
  --gas-prices 7aCC \
  --gas 250000 \
  --website="" -y
```

## Delete node
```python
sudo systemctl stop cascadiad
sudo systemctl disable cascadiad
rm /etc/systemd/system/cascadiad.service
sudo systemctl daemon-reload
cd $HOME
rm -rf cascadia
rm -rf .cascadiad
rm -rf $(which cascadiad)
```
#
### Sync Info
```python
cascadiad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
cascadiad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u cascadiad -f -o cat
```
### Check Balance
```python
cascadiad query bank balances cascadiad...addressjkl1yjgn7z09ua9vms259j
```
