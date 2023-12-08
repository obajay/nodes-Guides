# KiChain Mainnet guide

![Kic](https://github.com/obajay/nodes-Guides/assets/44331529/c98292e9-3c46-446c-82eb-aa6ffd4c8a75)

[WebSite](https://www.foundation.ki/)\
[GitHub](https://github.com/KiFoundation/ki-tools)
=
[EXPLORER](https://explorer.stavr.tech/Kichain-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
SOOON
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
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

# Build 26.05.23
```python
cd $HOME
git clone https://github.com/KiFoundation/ki-tools.git
cd ki-tools
git checkout 5.0.2
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`kid version --long | head`
- version: Mainnet-5.0.2
- commit: 17b36b312b2866831b1a9edff15745fe16e8d4cd

```python
kid init STAVRguide --chain-id kichain-2
kid config chain-id kichain-2
```    

## Create/recover wallet
```python
kid keys add <walletname>
            OR
kid keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.kid/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Kichain/genesis.json"

```
`sha256sum $HOME/.kid/config/genesis.json`
+ f42e1d49ca30f69ace60f5eb61416e9393d318083849e83d1fc33df4085462c0

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025uxki\"/;" ~/.kid/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.kid/config/config.toml
peers="6ebceb16da1c15c48086eaed1be807939f3e50f0@65.108.199.222:21636"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.kid/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.kid/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.kid/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.kid/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.kid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.kid/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.kid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.kid/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.kid/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.kid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Kichain/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/kid.service > /dev/null <<EOF
[Unit]
Description=kid
After=network-online.target

[Service]
User=$USER
ExecStart=$(which kid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync kichain Mainnet
```python
SOON
```
# SnapShot Mainnet (~3GB) updated every 7 hours  
```python
SOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable kid
sudo systemctl restart kid && sudo journalctl -u kid -f -o cat
```

### Create validator
```python
kid tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000uxki \
--pubkey $(kid tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id kichain-2 \
--fees="5000uxki" \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop kid
sudo systemctl disable kid
rm /etc/systemd/system/kid.service
sudo systemctl daemon-reload
cd $HOME
rm -rf ki-tools
rm -rf .kid
rm -rf $(which kid)
```
#
### Sync Info
```python
kid status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
kid status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u kid -f -o cat
```
### Check Balance
```python
kid query bank balances ki...addressjkl1yjgn7z09ua9vms259j
```
