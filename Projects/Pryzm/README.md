# Pryzm Testnet guide


![pry](https://github.com/obajay/nodes-Guides/assets/44331529/6af23090-51c3-40f0-b4e4-fede7c542d1e)

[DOCS](https://docs.pryzm.zone/)\
[GitHub](https://github.com/pryzm-finance)
=
[EXPLORER](https://explorer.stavr.tech/Pryzm-Testnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O pryzmt https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Pryzm/pryzmt && chmod +x pryzmt && ./pryzmt
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

# Build 11.01.24
```python
cd $HOME
wget -O pryzmd https://storage.googleapis.com/pryzm-zone/core/0.11.1/pryzmd-0.11.1-linux-amd64
chmod +x $HOME/pryzmd
mv pryzmd /root/go/bin/

```
*******🟢UPDATE🟢******* 11.01.24
```python
cd $HOME
wget -O pryzmd https://storage.googleapis.com/pryzm-zone/core/0.11.1/pryzmd-0.11.1-linux-amd64
chmod +x $HOME/pryzmd
mv pryzmd $(which pryzmd)
pryzmd version --long | grep -e version -e commit
#commit: dde07fd77b621d84403fb97ddbb118999e1fd186
#version: 0.11.1
sudo systemctl restart pryzmd && sudo journalctl -u pryzmd -f -o cat
```

`pryzmd version --long | grep -e version -e commit`
- version: 0.11.1
- commit: dde07fd77b621d84403fb97ddbb118999e1fd186

```python
pryzmd init STAVR_guide --chain-id indigo-1
pryzmd config chain-id indigo-1
```    

## Create/recover wallet
```python
pryzmd keys add <walletname>
  OR
pryzmd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.pryzm/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Pryzm/genesis.json"

```
`sha256sum $HOME/.pryzm/config/genesis.json`
+ 61c881fa75269f470d3abc34097a34bd6bd2f80ad5e7ecabaae8e4f9af4e2dcb

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.015upryzm, 0.01factory/pryzm15k9s9p0ar0cx27nayrgk6vmhyec3lj7vkry7rx/uusdsim,0.001ibc/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2,0.001ibc/265435C653FE85CD659E88CD51D4A735BDD4D3804871400378A488C71D68C72B,0.001ibc/92E0120F15D037353CFB73C14651FC8930ADC05B93100FD7754D3A689E53B333,0.001ibc/1704820C9E1F4A9925E0F23D3B92ED0E53DEE28726257E39FABD444BFC6B6AE3"|g' $HOME/.pryzm/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.pryzm/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.pryzm/config/config.toml
seeds="ff17ca4f46230306412ff5c0f5e85439ee5136f0@testnet-seed.pryzm.zone:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.pryzm/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.pryzm/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.pryzm/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.pryzm/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.pryzm/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.pryzm/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.pryzm/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.pryzm/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.pryzm/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Pryzm/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/pryzmd.service > /dev/null <<EOF
[Unit]
Description=pryzmd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which pryzmd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Pryzm Testnet
```python
SOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable pryzmd
sudo systemctl restart pryzmd && sudo journalctl -u pryzmd -f -o cat
```

[FAUCET](https://testnet.pryzm.zone/faucet)
=

### Create validator
```python
pryzmd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000upryzm \
--pubkey $(pryzmd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id indigo-1 \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.015upryzm \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop pryzmd
sudo systemctl disable pryzmd
rm /etc/systemd/system/pryzmd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .pryzm
rm -rf $(which pryzmd)
```
#
### Sync Info
```python
pryzmd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
pryzmd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u pryzmd -f -o cat
```
### Check Balance
```python
pryzmd query bank balances pryzm...addressjkl1yjgn7z09ua9vms259j
```
