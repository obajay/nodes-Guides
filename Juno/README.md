# Juno Mainnet guide

![Althea_Logo-BLUE_SIGNAL](https://user-images.githubusercontent.com/44331529/218240936-c2095305-1a28-45f6-8ccd-d068a4fe5754.svg)

[WebSite](https://www.althea.net/)\
[GitHub](https://github.com/CosmosContracts)
=
[EXPLORER 1](https://explorer.stavr.tech/juno/staking) \
[EXPLORER 2](https://ping.pub/juno/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  16| 32GB | 250GB    |


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

## GO 1.19.5
```python
ver="1.19.5" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 09.02.23
```python
cd $HOME
git clone https://github.com/CosmosContracts/juno juno
cd juno
git checkout v12.0.0
make install
```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`althea version --long`
- version: v2.0.0
- commit: c5286524143747a3a6df42ef5095a588b78df734

```python
junod init STAVRguide --chain-id juno-1
junod config chain-id juno-1
```    

## Create/recover wallet
```python
junod keys add <walletname>
  OR
junod keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.juno/config/genesis.json "SOON"

```
`sha256sum $HOME/.juno/config/genesis.json`
+ a5c08e53aca0390c45def85a6d16c0e7176bd0026b0a465aff5d1896ec0134a1

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ujuno\"/;" ~/.juno/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.althea/juno/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.juno/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.juno/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:12656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.juno/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.juno/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.juno/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.juno/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.juno/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.juno/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.juno/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.juno/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.juno/config/addrbook.json "SOON"
```

# Create a service file
```python
sudo tee /etc/systemd/system/junod.service > /dev/null <<EOF
[Unit]
Description=juno
After=network-online.target

[Service]
User=$USER
ExecStart=$(which junod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Althea Testnet
```python
soon
```
# SnapShot Mainnet (~0.2GB) updated every 5 hours  
```python
soon
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable junod
sudo systemctl restart junod && sudo journalctl -u junod -f -o cat
```

### Create validator
```python
junod tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000ujuno \
--pubkey $(junod tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id juno-1 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop junod && \
sudo systemctl disable junod && \
rm /etc/systemd/system/junod.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf juno && \
rm -rf .juno && \
rm -rf $(which junod)
```
#
### Sync Info
```python
junod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
junod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u junod -f -o cat
```
### Check Balance
```python
junod query bank balances juno...addressjkl1yjgn7z09ua9vms259j
```
