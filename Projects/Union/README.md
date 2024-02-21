# Union Testnet guide

![union](https://github.com/obajay/nodes-Guides/assets/44331529/ce76083b-17e7-4928-bffc-60f989b47ef3)

[WebSite](https://union.build/)
=
[EXPLORER](https://explorer.stavr.tech/Union-Testnet)
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

# Build 14.02.24
```python
cd $HOME
mkdir -p $HOME/go/bin/
wget https://uniont.binary.stavr.tech/union/uniond
chmod +x uniond
mv uniond $HOME/go/bin/

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```
## Init
```python
uniond --home $HOME/.union init STAVR_guide --chain-id union-testnet-6
```

`uniond --home $HOME/.union version --long`
- version: v0.19.0
- commit: d2a204a0b8d86b82a1bdeefbaeda90dc59b67673

## Create/recover wallet
```python
uniond --home $HOME/.union keys add <walletname>
            OR
uniond --home $HOME/.union keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.union/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Union/genesis.json"

```
`sha256sum $HOME/.union/config/genesis.json`
+ 1ab6b66403f9ae3b298ab68d4cac51e33d4c43a5b21fbe5d15b7bf2ac12a74b6

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0muno\"/;" ~/.union/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.union/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.union/config/config.toml
seeds="b37de4c50e26f7cde4c7b6ce06046a6693ffef2c@union.testnet.4.seed.poisonphang.com:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.union/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.union/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.union/config/config.toml

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
ExecStart=$(which uniond) start --home /root/.union
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
#pubkey
uniond tendermint show-validator --home /root/.union
cd $HOME
nano validator.json
{
  "pubkey": {#pubkey},
  "amount": "1000000muno",
  "moniker": "STAVR_Guide",
  "identity": "",
  "website": "",
  "security": "",
  "details": "",
  "commission-rate": "0.1",
  "commission-max-rate": "0.2",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}

uniond --home /root/.union tx staking create-validator /root/validator.json --from Wallet_name  --chain-id union-testnet-6
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
curl -s localhost:26657/status
```
### NodeINfo
```python
uniond --home $HOME/.union query staking validator unionvaloper155.........
```
### Check node logs
```python
sudo journalctl -u uniond -f -o cat
```
### Check Balance
```python
uniond query bank balances union...addressjkl1yjgn7z09ua9vms259j --home /.union
```
