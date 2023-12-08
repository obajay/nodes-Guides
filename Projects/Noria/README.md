# Noria Testnet guide

![noria](https://github.com/obajay/nodes-Guides/assets/44331529/3b6b03d6-58c9-404b-9d48-b79971af4b8f)

[WebSite](https://noria.network/)\
[GitHub](https://github.com/noria-net)
=
[EXPLORER](https://explorer.stavr.tech/Noria-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O noriat https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Noria/noriat && chmod +x noriat && ./noriat
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
git clone https://github.com/noria-net/noria.git
cd noria
git checkout v1.3.0
make install
```
*******🟢UPDATE🟢******* 16.06.23
```python
cd noria
git fetch --all
git checkout v1.3.0
make install
noriad version --long | grep -e commit -e version
#commit: 1533a05c67efd5e0c36269811de3c86ecb51ab96
#version: 1.3.0
sudo systemctl restart noriad && sudo journalctl -u noriad -f -o cat
```

`noriad version --long | grep -e commit -e version`
- version: 1.3.0
- commit: 1533a05c67efd5e0c36269811de3c86ecb51ab96

```python
noriad init STAVRguide --chain-id oasis-3
noriad config chain-id oasis-3
```    

## Create/recover wallet
```python
noriad keys add <walletname>
  OR
noriad keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/noria-net/noria/main/genesis.json -O $HOME/.noria/config/genesis.json

```
`sha256sum $HOME/.noria/config/genesis.json`
+ 258f6d5fc4ebbef308bcd7f0ae85407342e6c1a95ce39a72b625adaa4c4a0c46

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ucrd\"/;" ~/.noria/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.noria/config/config.toml
peers="6b00a46b8c79deab378a8c1d5c2a63123b799e46@34.69.0.43:26656,4d8147a80c46ba21a8a276d55e6993353e03a734@165.22.42.220:26656,e82fb793620a13e989be8b2521e94db988851c3c@165.227.113.152:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noria/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.noria/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.noria/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.noria/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.noria/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.noria/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.noria/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.noria/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.noria/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.noria/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Noria/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/noriad.service > /dev/null <<EOF
[Unit]
Description=noria
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noriad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Testnet
```python
SOOOOOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOOOOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable noriad
sudo systemctl restart noriad && sudo journalctl -u noriad -f -o cat
```

### Create validator
```python
noriad tx staking create-validator \
--amount=1000000unoria \
--pubkey=$(noriad tendermint show-validator) \
--moniker=STAVRguide \
--chain-id=oasis-3 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--from=wallet \
--details="" \
--identity="" \
--fees 500ucrd \
--website="" -y
```

## Delete node
```python
sudo systemctl stop noriad && \
sudo systemctl disable noriad && \
rm /etc/systemd/system/noriad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf noria && \
rm -rf .noria && \
rm -rf $(which noriad)
```
#
### Sync Info
```python
noriad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
noriad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u noriad -f -o cat
```
### Check Balance
```python
noriad query bank balances noria...addressjkl1yjgn7z09ua9vms259j
```
