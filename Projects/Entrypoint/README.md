# Entrypoint Testnet guide

![ennerty](https://github.com/obajay/nodes-Guides/assets/44331529/fb4b2857-b9c6-4935-b673-28efb8f28a70)

[WebSite](https://entrypoint.zone/)\
[GitHub](https://github.com/entrypoint-zone/testnets/tree/main/entrypoint-pubtest-2)
=
[EXPLORER](https://explorer.stavr.tech/Entrypoint-Testnet)
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

# Build 06.12.23
```python
cd $HOME
wget -O entrypointd https://github.com/entrypoint-zone/testnets/releases/download/v1.3.0/entrypointd-1.3.0-linux-amd64
chmod +x entrypointd
mv entrypointd $HOME/go/bin/

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`entrypointd version --long | head`
- version: 1.3.0
- commit: 5f0d3570949695395f084fc3d4137c0ec70665a0

```python
entrypointd init STAVR_guide --chain-id entrypoint-pubtest-2
entrypointd config chain-id entrypoint-pubtest-2
```    

## Create/recover wallet
```python
entrypointd keys add <walletname>
            OR
entrypointd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.entrypoint/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Entrypoint/genesis.json"

```
`sha256sum $HOME/.entrypoint/config/genesis.json`
+ 15ff4506d0a1a4fec33a0fef6a5d19be3c4dfbe692f1f71153dcb3eb2a2be06b

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uentry\"/;" ~/.entrypoint/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.entrypoint/config/config.toml
peers="81bf2ade773a30eccdfee58a041974461f1838d8@185.107.68.148:26656,d57c7572d58cb3043770f2c0ba412b35035233ad@80.64.208.169:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.entrypoint/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.entrypoint/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.entrypoint/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.entrypoint/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.entrypoint/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.entrypoint/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.entrypoint/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.entrypoint/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.entrypoint/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.entrypoint/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Entrypoint/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/entrypointd.service > /dev/null <<EOF
[Unit]
Description=entrypointd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which entrypointd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Entrypoint Testnet
```python
SOOON
```
# SnapShot Mainnet (~3GB) updated every 7 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable entrypointd
sudo systemctl restart entrypointd && sudo journalctl -u entrypointd -f -o cat
```

### Create validator
```python
entrypointd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000uentry \
--pubkey $(entrypointd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id entrypoint-pubtest-2 \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop entrypointd
sudo systemctl disable entrypointd
rm /etc/systemd/system/entrypointd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .entrypoint
rm -rf $(which entrypointd)
```
#
### Sync Info
```python
entrypointd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
entrypointd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u entrypointd -f -o cat
```
### Check Balance
```python
entrypointd query bank balances entrypoint...addressjkl1yjgn7z09ua9vms259j
```
