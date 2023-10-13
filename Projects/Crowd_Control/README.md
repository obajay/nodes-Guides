# Crowd Control testnet guide

![Crowd Control](https://user-images.githubusercontent.com/44331529/180597315-e25b1929-8973-4149-b2c6-b9086c1787bd.png)

[EXPLORER 1](http://explorer.stavr.tech/CARDCHAIN/staking) \
[EXPLORER 2](https://explorer.bccnodes.com/cardchain/staking)
=
- **Minimum hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   2| 4GB  | 100GB    |

# 1) Auto_install script 
```python
wget -O crowd https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crowd_Control/crowd && chmod +x crowd && ./crowd
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.20.5 (one command)
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

# Build 13.10.23
```python
git clone https://github.com/DecentralCardGame/Cardchain
wget https://github.com/DecentralCardGame/Cardchain/releases/download/v0.9.0/Cardchaind
chmod +x Cardchaind
mv $HOME/Cardchaind /usr/local/bin
```
`Cardchaind version --long | grep -e commit -e version`
+ version: 0.9.0
+ commit: 96dd28d01934b3ed6ead27456313167b2cef3f76
    
# Init node and download Genesis
```python
Cardchaind init STAVRguide --chain-id cardtestnet-4
Cardchaind config chain-id cardtestnet-4
wget http://45.136.28.158:3000/genesis.json -O $HOME/.Cardchain/config/genesis.json
```
## Create/recover wallet
```python
Cardchaind keys add <walletname>
Cardchaind keys add <walletname> --recover
```

## Download addrbook
```python
wget -O $HOME/.Cardchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crowd_Control/addrbook.json"
```

## Minimum gas price/Peers/Seeds
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ubpf\"/;" ~/.Cardchain/config/app.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.Cardchain/config/config.toml
peers="1ed98c796bcdd0faf5a7ad8793d229e3c7d89543@lxgr.xyz:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.Cardchain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.Cardchain/config/config.toml
```


### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="1000" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain/config/app.toml
```
### Indexer (optional)
```python
ndexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.Cardchain/config/config.toml
```

# Service file
```python
sudo tee <<EOF >/dev/null /etc/systemd/system/Cardchaind.service
[Unit]
Description=Cardchain Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which Cardchaind) start
Restart=always
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
## StateSync 
```python

```

## SnapShot (~0.3 GB) updated every 5 hours
```python

```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable Cardchaind
sudo systemctl restart Cardchaind && sudo journalctl -u Cardchaind -f -o cat
```

## Create validator
```python
Cardchaind tx staking create-validator \
--amount 1000000ubpf \
--from <walletName> \
--commission-max-change-rate "0.2" \
--commission-max-rate "1" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--details="" \
--identity="" \
--pubkey  $(Cardchaind tendermint show-validator) \
--moniker STAVR_Guide \
--fees 300ubpf \
--chain-id cardtestnet-4 -y
```

## Delete node
```python
sudo systemctl stop Cardchaind
sudo rm /etc/systemd/system/Cardchaind.service
sudo rm -rf $HOME/.Cardchain/
sudo rm -rf Testnet
sudo rm -rf $(which Cardchaind)
```
