# eMoney Mainnet guide

![emon](https://github.com/obajay/nodes-Guides/assets/44331529/77ea4a55-b118-43de-a286-f92c5717655f)


[WebSite](https://e-money.com/) \
[GitHub](https://github.com/e-money)
=
[EXPLORER 1](https://explorer.stavr.tech/eMoney-Mainnet/) \
[EXPLORER 2](https://ping.pub/e-money/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
SOON
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
git clone https://github.com/e-money/em-ledger/
cd em-ledger
git checkout v1.2.0
make build && cd build
mv emd $HOME/go/bin/

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`emd version --long | grep -e commit -e version`
- version: v1.2.0
- commit: 601cd6c1dcd6d1e1ce8e9d795b6ac56a44c66deb

```python
emd init STAVR_guide --chain-id emoney-3
emd config chain-id emoney-3
```    

## Create/recover wallet
```python
emd keys add <walletname>
  OR
emd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.emd/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/eMoney/genesis.json"
```
`sha256sum $HOME/.emd/config/genesis.json`
+ a9d7982c5159ec41f7eeb046693433669c96dc6a86dce8b3938134c35c565e79

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ungm\"/;" ~/.emd/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.emd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.emd/config/config.toml
peers="ecec8933d80da5fccda6bdd72befe7e064279fc1@207.180.213.123:26676,0ad7bc7687112e212bac404670aa24cd6116d097@50.18.83.75:26656,1723e34f45f54584f44d193ce9fd9c65271ca0b3@13.124.62.83:26656,34eca4a9142bf9c087a987b572c114dad67a8cc5@172.105.148.191:26656,0b186517e4d82eb4c000a567e486b7b96bf19752@44.195.95.22:26656,c2766f7f6dfe95f2eb33e99a538acf3d6ec608b1@162.55.132.230:2140"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.emd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.emd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.emd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.emd/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.emd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.emd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.emd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.emd/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.emd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.emd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/eMoney/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/emd.service > /dev/null <<EOF
[Unit]
Description=emd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which emd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync eMoney Mainnet
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
sudo systemctl enable emd
sudo systemctl restart emd && sudo journalctl -u emd -f -o cat
```

### Create validator
```python
emd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.1 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000ungm \
--pubkey $(emd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id emoney-3 \
--gas 300000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/eMoney)
=


## Delete node
```python
sudo systemctl stop emd
sudo systemctl disable emd
rm /etc/systemd/system/emd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf em-ledger
rm -rf .emd
rm -rf $(which emd)
```
#
### Sync Info
```python
emd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
emd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u emd -f -o cat
```
### Check Balance
```python
emd query bank balances emoney...addressjkl1yjgn7z09ua9vms259j
```
