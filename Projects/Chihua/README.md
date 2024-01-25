# Chihuahua Mainnet guide

![chihua](https://github.com/obajay/nodes-Guides/assets/44331529/fde200a3-1f5d-46f1-b13f-f07c4d49fe3d)

[WebSite](https://www.chihuahua.wtf/) \
[GitHub](https://github.com/ChihuahuaChain/chihuahua)
=
[EXPLORER 1](https://explorer.stavr.tech/Chihua-Mainnet) \
[EXPLORER 2](https://ping.pub/chihuahua/)
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

## GO 1.21.4
```python
ver="1.21.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 23.12.23
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/ChihuahuaChain/chihuahua chihuahua
cd chihuahua
git checkout v6.0.1
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`chihuahuad version --long | grep -e commit -e version`
- version: v6.0.1
- commit: eb1bc83d8ba715e38a2b24fedeaebb624259827a

```python
chihuahuad init STAVR_guide --chain-id chihuahua-1
chihuahuad config chain-id chihuahua-1
```    

## Create/recover wallet
```python
chihuahuad keys add <walletname>
  OR
chihuahuad keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.chihuahuad/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Chihua/genesis.json"
```
`sha256sum $HOME/.chihuahuad/config/genesis.json`
+ 200a64f201c6b5799d81bcf52a25ce4eb1c0eac3f7c8c5eaa8335e75c5763f91

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uhuahua\"/;" ~/.chihuahuad/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.chihuahuad/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.chihuahuad/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.chihuahuad/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:12956"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.chihuahuad/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.chihuahuad/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.chihuahuad/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.chihuahuad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.chihuahuad/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.chihuahuad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.chihuahuad/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.chihuahuad/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.chihuahuad/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Chihua/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/chihuahuad.service > /dev/null <<EOF
[Unit]
Description=chihuahuad
After=network-online.target

[Service]
User=$USER
ExecStart=$(which chihuahuad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Chihuahua Mainnet
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
sudo systemctl enable chihuahuad
sudo systemctl restart chihuahuad && sudo journalctl -u chihuahuad -f -o cat
```

### Create validator
```python
chihuahuad tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 100000000uhuahua \
--pubkey $(chihuahuad tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id chihuahua-1 \
--gas 300000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Chihua)
=


## Delete node
```python
sudo systemctl stop chihuahuad
sudo systemctl disable chihuahuad
rm /etc/systemd/system/chihuahuad.service
sudo systemctl daemon-reload
cd $HOME
rm -rf chihuahua
rm -rf .chihuahuad
rm -rf $(which chihuahuad)
```
#
### Sync Info
```python
chihuahuad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
chihuahuad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u chihuahuad -f -o cat
```
### Check Balance
```python
chihuahuad query bank balances chihuahua...addressjkl1yjgn7z09ua9vms259j
```
