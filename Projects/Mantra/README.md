# Mantra Testnet guide

[EXPLORER](https://explorer.stavr.tech/Mantra-Testnet/staking)
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

# Build 13.11.23
```python
cd $HOME
sudo wget -O /usr/lib/libwasmvm.x86_64.so https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so
wget https://testnet-files.itrocket.net/mantra/mantrachaind-linux-amd64.zip
unzip mantrachaind-linux-amd64.zip
rm mantrachaind-linux-amd64.zip
mv mantrachaind $HOME/go/bin

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`mantrachaind version --long | grep -e commit -e version`
- version: 1.0.0
- commit: 

```python
mantrachaind init STAVRguide --chain-id mantrachain-testnet-1
mantrachaind config chain-id mantrachain-testnet-1
```    

## Create/recover wallet
```python
mantrachaind keys add <walletname>
  OR
mantrachaind keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.mantrachain/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Mantra/genesis.json"

```
`sha256sum $HOME/.mantrachain/config/genesis.json`
+ 119f80b7705de79ffffeb29fc5374e45fdd3f45968b5acee2c062018bcb9f70b

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0001uaum\"/;" ~/.mantrachain/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.mantrachain/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mantrachain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.mantrachain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 10/g' $HOME/.mantrachain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 40/g' $HOME/.mantrachain/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.mantrachain/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mantrachain/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.mantrachain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Mantra/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/mantrachaind.service > /dev/null <<EOF
[Unit]
Description=mantrachaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which mantrachaind) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Testnet
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
sudo systemctl enable mantrachaind
sudo systemctl restart mantrachaind && sudo journalctl -u mantrachaind -f -o cat
```

### Create validator
```python
mantrachaind tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000uaum \
--pubkey $(mantrachaind tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id mantrachain-testnet-1 \
--fees 2020uaum \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop mantrachaind
sudo systemctl disable mantrachaind
rm /etc/systemd/system/mantrachaind.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .mantrachain
rm -rf $(which mantrachaind)
```
#
### Sync Info
```python
mantrachaind status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
mantrachaind status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u mantrachaind -f -o cat
```
### Check Balance
```python
mantrachaind query bank balances mantra...addressjkl1yjgn7z09ua9vms259j
```
