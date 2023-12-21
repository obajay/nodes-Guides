# Union Testnet guide

![union](https://github.com/obajay/nodes-Guides/assets/44331529/ce76083b-17e7-4928-bffc-60f989b47ef3)

[WebSite](https://union.build/)
=
[EXPLORER](https://explorer.stavr.tech/Shentu-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O uniont https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Union/uniont && chmod +x uniont && ./uniont
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

# Build 21.12.23
```python
cd $HOME
wget http://uniont.binary.stavr.tech:15/union/uniond
chmod +x uniond
mv uniond $HOME/go/bin/

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`uniond version --long`
- version: v0.17.0
- commit: fa6dfecb6cfd89b6827c2992efb37675f8a147ab

```python
uniond init STAVR_guide --chain-id union-testnet-4
uniond config chain-id union-testnet-4
```    

## Create/recover wallet
```python
uniond keys add <walletname>
            OR
uniond keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.union/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Union/genesis.json"

```
`sha256sum $HOME/.union/config/genesis.json`
+ 93d05b1e71007e2d5c82676418b4ca632834b45e10e1ef9129d4d8a54275a0cb

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0muno\"/;" ~/.union/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.union/config/config.toml
peers="b2f2c6ba26958a1daf5838dee130fe0f0d75518d@34.171.89.160:26656,728ec2b975df5d9afc7bc98f8b74ba7161d2955b@65.109.29.23:26656,7c743b507ec3b67bc790c826ec471d2635c992f7@88.99.3.158:24656,8b13facd07099883a2275db870834390109ded62@92.243.27.215:27656,821eade3cdada32cd15bfc7bd941e5bfad173d35@5.9.115.189:26656,b82b7b8d739c0869b0c2c369770a3adf66a126cb@198.244.179.173:26656,65d3fbc95488503554d554f6332db4dbd68accb0@65.109.69.239:15007"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.union/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.union/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.union/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.union/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.union/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.union/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.union/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.union/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.union/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.union/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Union/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/uniond.service > /dev/null <<EOF
[Unit]
Description=uniond
After=network-online.target

[Service]
User=$USER
ExecStart=$(which uniond) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Union Testnet
```python
SOOON
```
# SnapShot
```python
SOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable uniond
sudo systemctl restart uniond && sudo journalctl -u uniond -f -o cat
```

### Create validator
```python
uniond tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount="1000000"muno \
--pubkey $(uniond tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id union-testnet-4 \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop uniond
sudo systemctl disable uniond
rm /etc/systemd/system/uniond.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .union
rm -rf $(which uniond)
```
#
### Sync Info
```python
uniond status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
uniond status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u uniond -f -o cat
```
### Check Balance
```python
uniond query bank balances union...addressjkl1yjgn7z09ua9vms259j
```
