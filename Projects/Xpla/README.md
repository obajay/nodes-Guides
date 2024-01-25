# XPLA Mainnet guide

![xpla](https://github.com/obajay/nodes-Guides/assets/44331529/31888016-4422-4fd3-97ff-4b105bf9adb2)

[WebSite](https://xpla.io) \
[GitHub](https://github.com/xpladev/xpla)
=
[EXPLORER 1](https://explorer.stavr.tech/Xpla-Mainnet/) \
[EXPLORER 2](https://explorer.xpla.io/mainnet/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O xplam https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Xpla/xplam && chmod +x xplam && ./xplam
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.20.5
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

# Build 18.12.23
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/xpladev/xpla
cd xpla
git checkout v1.3.3
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`xplad version --long | grep -e commit -e version`
- version: v1.3.3
- commit: 263e93fa7cc065deda5ba5c172ffff5db9124327

```python
xplad init STAVR_guide --chain-id dimension_37-1
xplad config chain-id dimension_37-1
```    

## Create/recover wallet
```python
xplad keys add <walletname>
  OR
xplad keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.xpla/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Xpla/genesis.json"
```
`sha256sum $HOME/.xpla/config/genesis.json`
+ 13c41071c3eccac14e8b2292006e8bc4d2a71d2acb6871a99dc9fc4a569ce1c0

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.1axpla\"/;" $HOME/.xpla/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.xpla/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.xpla/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.xpla/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:20156"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.xpla/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.xpla/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.xpla/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.xpla/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.xpla/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.xpla/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.xpla/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.xpla/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.xpla/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Xpla/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/xplad.service > /dev/null <<EOF
[Unit]
Description=xplad
After=network-online.target

[Service]
User=$USER
ExecStart=$(which xplad) start --home /root/.xpla
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Xpla Mainnet
```python
SOOON
```
# SnapShot Mainnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable xplad
sudo systemctl restart xplad && sudo journalctl -u xplad -f -o cat
```

### Create validator
```python
xplad tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.1 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000000000000000axpla \
--pubkey $(xplad tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id dimension_37-1 \
--gas 300000 \
--fees 255000000000000000axpla \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Xpla)
=


## Delete node
```python
sudo systemctl stop xplad
sudo systemctl disable xplad
rm /etc/systemd/system/xplad.service
sudo systemctl daemon-reload
cd $HOME
rm -rf xpla
rm -rf .xpla
rm -rf $(which xplad)
```
#
### Sync Info
```python
xplad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
xplad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u xplad -f -o cat
```
### Check Balance
```python
xplad query bank balances xpla...addressjkl1yjgn7z09ua9vms259j
```
