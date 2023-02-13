# AndromedaProtocol Testnet guide

![andro](https://user-images.githubusercontent.com/44331529/218533292-7846187a-3897-46ad-9cc8-f17cc18253d3.png)

[WebSite](https://andromedaprotocol.io/)\
[GitHub](https://github.com/andromedaprotocol)
=
[EXPLORER 1](https://explorer.stavr.tech/andromedad-testnet/staking)\
[EXPLORER 2](https://testnet-ping.wildsage.io/andromeda/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
SOON
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
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad
git checkout galileo-3-v1.1.0-beta1 
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`andromedad version --long`
- version: galileo-3-v1.1.0-beta1
- commit: b3f8d880dfcdb3265d321e465b47b04071d9480f

```python
andromedad init STAVRguide --chain-id galileo-3
andromedad config chain-id galileo-3
```    

## Create/recover wallet
```python
andromedad keys add <walletname>
  OR
andromedad keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.andromedad/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/AndromedaProtocol/genesis.json"

```
`sha256sum $HOME/.andromedad/config/genesis.json`
+ 26cefef408b9cdc013e7427d5fb05bbd44a006d80b7e0db36aaf125edda9b4e6

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uandr\"/;" ~/.andromedad/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.andromedad/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.althea/config/config.toml
peers="06d4ab2369406136c00a839efc30ea5df9acaf11@10.128.0.44:26656,43d667323445c8f4d450d5d5352f499fa04839a8@192.168.0.237:26656,29a9c5bfb54343d25c89d7119fade8b18201c503@192.168.101.79:26656,6006190d5a3a9686bbcce26abc79c7f3f868f43a@37.252.184.230:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.andromedad/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.andromedad/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.andromedad/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.andromedad/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.andromedad/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.andromedad/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.andromedad/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/AndromedaProtocol/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/andromedad.service > /dev/null <<EOF
[Unit]
Description=andromedad
After=network-online.target

[Service]
User=$USER
ExecStart=$(which andromedad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Andromedad Testnet
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
sudo systemctl enable andromedad
sudo systemctl restart andromedad && sudo journalctl -u andromedad -f -o cat
```

### Create validator
```python
andromedad tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000uandr \
--pubkey $(andromedad tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id galileo-3 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop andromedad && \
sudo systemctl disable andromedad && \
rm /etc/systemd/system/andromedad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf andromedad && \
rm -rf .andromedad && \
rm -rf $(which andromedad)
```
#
### Sync Info
```python
andromedad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
andromedad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u andromedad -f -o cat
```
### Check Balance
```python
andromedad query bank balances andr...addressjkl1yjgn7z09ua9vms259j
```
