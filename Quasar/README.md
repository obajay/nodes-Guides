# Quasar Testnet guide


![wuasar](https://user-images.githubusercontent.com/44331529/219620351-e5199eac-475e-4fc8-b851-242d7fbf9643.png)

[WebSite](https://www.quasar.fi/)\
[GitHub](https://github.com/quasar-finance)
=
[EXPLORER 1](https://explorer.stavr.tech/quasar-mainnet/staking) \
[EXPLORER 2](https://www.mintscan.io/quasar/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O quasar https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quasar/quasar && chmod +x quasar && ./quasar
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

# Build 12.02.23
```python
cd $HOME
wget https://github.com/quasar-finance/binary-release/raw/main/v0.0.2-alpha-11/quasarnoded-linux-amd64
chmod +x quasarnoded-linux-amd64
mkdir go && mkdir go/bin
mv quasarnoded-linux-amd64 $HOME/go/bin/quasarnoded
```

*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`quasarnoded version --long`
- commit: c426178652265b29f7c3ea05b6e144542584e523
- version: 0.0.2-alpha-11

```python
quasarnoded init STAVRguide --chain-id qsr-questnet-04
quasarnoded config chain-id qsr-questnet-04
```    

## Create/recover wallet
```python
quasarnoded keys add <walletname>
  OR
quasarnoded keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.quasarnode/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quasar/genesis.json"
```
`sha256sum $HOME/.quasarnode/config/genesis.json`
+ 8f38e35f88f4cbe983f7791d0d49b3f4123660c472408892c46ab145855fe3a5

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uqsr\"/;" ~/.quasarnode/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.quasarnode/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.quasarnode/config/config.toml
peers="8a19aa6e874ed5720aad2e7d02567ec932d92d22@141.94.248.63:26656,444b80ce750976df59b88ac2e08d720e1dbbf230@68.183.75.239:26666,20b4f9207cdc9d0310399f848f057621f7251846@222.106.187.13:40606"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.quasarnode/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.quasarnode/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.quasarnode/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.quasarnode/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.quasarnode/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.quasarnode/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.quasarnode/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.quasarnode/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.quasarnode/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.quasarnode/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quasar/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/quasarnoded.service > /dev/null <<EOF
[Unit]
Description=quasarnoded
After=network-online.target

[Service]
User=$USER
ExecStart=$(which quasarnoded) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Quasar Testnet
```python
SOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable quasarnoded
sudo systemctl restart quasarnoded && sudo journalctl -u quasarnoded -f -o cat
```

### Create validator
```python
quasarnoded tx staking create-validator \
  --amount=1000000uqsr \
  --pubkey=$(quasarnoded tendermint show-validator) \
  --moniker="STAVRguide" \
  --details="" \
  --identity="" \
  --website="" \
  --chain-id="qsr-questnet-04" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=WalletName -y
```

## Delete node
```python
sudo systemctl stop quasarnoded && \
sudo systemctl disable quasarnoded && \
rm /etc/systemd/system/quasarnoded.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .quasarnode && \
rm -rf $(which quasarnoded)
```
#
### Sync Info
```python
quasarnoded status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
quasarnoded status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u quasarnoded -f -o cat
```
### Check Balance
```python
quasarnoded query bank balances quasar...addressjkl1yjgn7z09ua9vms259j
```
