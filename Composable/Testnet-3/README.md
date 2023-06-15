# Composable Testnet-3 guide

![compo](https://github.com/obajay/nodes-Guides/assets/44331529/49502f93-cb03-461e-b788-78a391456f72)

[WebSite](https://www.composable.finance/)\
[GitHub](https://github.com/notional-labs/composable-networks)
=
[EXPLORER 1](https://explorer.nodestake.top/banksy-testnet/staking) \
[EXPLORER 2](https://explorer.nodexcapital.com/composable/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet-3   |   4|  8GB | 150GB    |


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

# Build 15.06.23
```python
cd $HOME
git clone https://github.com/notional-labs/composable-testnet
cd composable-testnet
git checkout v3.0.3-testnet
make build
cd bin
mv centaurid $HOME/go/bin/banksyd
```
*******🟢UPDATE🟢******* 15.06.23
```python
cd $HOME
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-centauri/
cd composable-centauri
git checkout v3.0.3-testnet
make build
cd bin
mv centaurid $HOME/go/bin/banksyd
banksyd version --long | grep -e commit -e version
#version: v3.0.3-testnet
#commit: 1bc799bd823dae4579bc925c51ede67a7411a43f
sudo systemctl restart banksyd && sudo journalctl -u banksyd -f -o cat
curl -s http://localhost:26657/consensus_state  | jq '.result.round_state.height_vote_set[0].prevotes_bit_array'

```

`banksyd version --long`
- version: v3.0.3-testnet
- commit: 1bc799bd823dae4579bc925c51ede67a7411a43f

```python
banksyd init STAVRguide --chain-id banksy-testnet-3
banksyd config chain-id banksy-testnet-3
```    

## Create/recover wallet
```python
banksyd keys add <walletname>
  OR
banksyd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O ~/.banksy/config/genesis.json https://raw.githubusercontent.com/notional-labs/composable-networks/main/banksy-testnet-3/genesis.json
```
`sha256sum $HOME/.banksy/config/genesis.json`
+ 1ff7de58f849f954b32bf3f565baf2548a3169a77111ba3c69cba25d020ce589

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0upica\"/;" ~/.banksy/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.banksy/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.banksy/config/config.toml
seeds="364b8245e72f083b0aa3e0d59b832020b66e9e9d@65.109.80.150:21500"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.banksy/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.banksy/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.banksy/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.banksy/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.banksy/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.banksy/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Composable/Testnet-3/addrbook.json"
```

# Create a service file
```python
tee /etc/systemd/system/banksyd.service > /dev/null <<EOF
[Unit]
Description=banksyd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which banksyd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Testnet
```python
SOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable banksyd
sudo systemctl restart banksyd && sudo journalctl -u banksyd -f -o cat
```

### Create validator
```python
banksyd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000upica \
--pubkey $(banksyd tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id banksy-testnet-3\
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop banksyd && \
sudo systemctl disable banksyd && \
rm /etc/systemd/system/banksyd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf composable-testnet && \
rm -rf .banksy && \
rm -rf $(which banksyd)
```
#
### Sync Info
```python
banksyd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
banksyd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u banksyd -f -o cat
```
### Check Balance
```python
banksyd query bank balances banksy...addressjkl1yjgn7z09ua9vms259j
```
