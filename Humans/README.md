# Humans Testnet guide

![humans](https://user-images.githubusercontent.com/44331529/214357205-c725ec73-6d62-4cbd-a847-b57247240a6a.png)


[WebSite](https://humans.ai/) \
[GitHub](https://github.com/humansdotai/humans)
=
[EXPLORER](https://explorer.stavr.tech/humans-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O hum https://raw.githubusercontent.com/obajay/nodes-Guides/main/Humans/hum && chmod +x hum && ./hum
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

# Build 18.05.23
```python
cd $HOME
git clone https://github.com/humansdotai/humans
cd humans && git checkout tags/v0.2.2
make install
```
`humansd version --long`
- version: 0.2.2
- commit: a3e608e8fc45ace7055fc312b7e5f4831ca79816

```python
humansd config chain-id humans_3000-23
humansd init STAVRguide --chain-id humans_3000-23
```    

## Create/recover wallet
```python
humansd keys add <walletname>
          or 
humansd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.humansd/config/genesis.json "https://raw.githubusercontent.com/humansdotai/testnets/master/friction/mission-2/genesis.json"

```
`sha256sum $HOME/.humansd/config/genesis.json`
+ be45acc413ef1ff73a19c796e74b84acdeb65b14d672684dc2374889c898cd3d

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.humans/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uheart\"/;" ~/.humans/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.humans/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.humans/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.humans/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.humans/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.humans/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.humans/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.humans/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Humans/addrbook.json"
```
# StateSync
```python
SOOON
```
# SnapShot (~0.2 GB) updated every 5 hours
```python
SOON
```

# Create a service file
```python
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=humans
After=network-online.target

[Service]
User=$USER
ExecStart=$(which humansd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload && sudo systemctl enable humansd
sudo systemctl restart humansd && sudo journalctl -u humansd -f -o cat
```

### Create validator
```python
humansd tx staking create-validator \
  --amount 1000000uheart \
  --from wallet \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(humansd tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id testnet-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop humansd && \
sudo systemctl disable humansd && \
rm /etc/systemd/system/humansd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf humans && \
rm -rf .humans && \
rm -rf $(which humansd)
```
#
### Sync Info
```python
humansd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
humansd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u humansd -f -o cat
```
### Check Balance
```python
humansd query bank balances humans...address1yjgn7z09ua9vms259j
```
