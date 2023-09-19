# Sao Beta guide

![Sao](https://user-images.githubusercontent.com/44331529/223452452-434790b5-0079-4402-b6f5-9ddc878ac826.png)

[WebSite](https://www.sao.network/#/)\
[GitHub]( https://github.com/SaoNetwork)
=
[EXPLORER 1](https://explorer.stavr.tech/Sao-Beta/staking) \
[EXPLORER 2](https://explorer.sao.network/sao-beta/staking)
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

# Build 09.09.23
```python
cd $HOME
git clone https://github.com/SaoNetwork/sao-consensus.git
cd sao-consensus
git checkout v0.1.8
make install
```
*******🟢UPDATE🟢******* 09.09.23
```python
cd $HOME/sao-consensus
git fetch
git checkout v0.1.8
make install
saod version --long | grep -e commit -e version
#version: 0.1.8
#commit: 1034bf8c1d81d02c32b1be38db45e58654a67935
sudo systemctl restart saod && sudo journalctl -u saod -f -o cat
```

`saod version --long | grep -e commit -e version`
- version: 0.1.8
- commit: 1034bf8c1d81d02c32b1be38db45e58654a67935

```python
saod init STAVRguide --chain-id sao-20230629
saod config chain-id sao-20230629
```    

## Create/recover wallet
```python
saod keys add <walletname>
  OR
saod keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.sao/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sao/Beta/genesis.json"

```
`sha256sum $HOME/.sao/config/genesis.json`
+ 6fb4ae6e153970fed712ac3e0d893a8c3fafa0e5d6f5c0ac7bbab0d068dbbf4c

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0usct\"/;" ~/.sao/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sao/config/config.toml
peers="b10aa9b0535d444b5ae9a670e74a265ed5f34590@8.222.242.30:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sao/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sao/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.sao/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.sao/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sao/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sao/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sao/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.sao/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sao/Beta/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/saod.service > /dev/null <<EOF
[Unit]
Description=sao
After=network-online.target

[Service]
User=$USER
ExecStart=$(which saod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Sao Beta
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
sudo systemctl enable saod
sudo systemctl restart saod && sudo journalctl -u saod -f -o cat
```

### Create validator
```python
saod tx staking create-validator \
  --amount=1000000usct \
  --pubkey=$(saod tendermint show-validator) \
  --moniker="STAVRguide" \
  --details="" \
  --identity="" \
  --website="" \
  --chain-id="sao-20230629" \
  --commission-rate="0.10" \
  --commission-max-rate="0.50" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=wallet \
  --node "tcp://localhost:1077" -y
```

## Delete node
```python
sudo systemctl stop saod && \
sudo systemctl disable saod && \
rm /etc/systemd/system/saod.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf sao-consensus && \
rm -rf .sao && \
rm -rf $(which saod)
```
#
### Sync Info
```python
saod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
saod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u saod -f -o cat
```
### Check Balance
```python
saod query bank balances saod...addressjkl1yjgn7z09ua9vms259j
```
