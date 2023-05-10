# Gitopia Mainnet guide

![gitopia](https://user-images.githubusercontent.com/44331529/200920964-90530a1f-8225-4021-923c-43712c00bb21.png)

[WebSite](https://gitopia.com/home)
=
[EXPLORER 1](https://explorer.stavr.tech/gitopia-testnet/staking) \
[EXPLORER 2](https://explorer.gitopia.com/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 150GB    |


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

# Build 10.05.23
```python
curl https://get.gitopia.com | bash
git clone -b v1.2.0 gitopia://gitopia/gitopia
cd gitopia && make install
```
`gitopiad version --long`
- version: v1.2.0

```python
gitopiad init STAVRguide --chain-id gitopia-janus-testnet-2 

```    

## Create/recover wallet
```python
gitopiad keys add <walletname>
gitopiad keys add <walletname> --recover
```

## Download Genesis
```python
wget https://server.gitopia.com/raw/gitopia/testnets/master/gitopia-janus-testnet-2/genesis.json.gz
gunzip genesis.json.gz
mv genesis.json $HOME/.gitopia/config/genesis.json
```
`sha256sum $HOME/.gitopia/config/genesis.json`
+ 038a81d821f3d8f99e782cbfed609e4853d24843c48a1469287528e632a26162

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utlore\"/" $HOME/.gitopia/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.gitopia/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.gitopia/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.gitopia/config/config.toml
seeds="399d4e19186577b04c23296c4f7ecc53e61080cb@seed.gitopia.com:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.gitopia/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.gitopia/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.gitopia/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.gitopia/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.gitopia/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.gitopia/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.gitopia/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.gitopia/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.gitopia/config/addrbook.json "SOOON"
```

## StateSync
```python
SOOON
```

# Create a service file
```python
sudo tee /etc/systemd/system/gitopiad.service > /dev/null <<EOF
[Unit]
Description=gitopia
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gitopiad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable gitopiad
sudo systemctl restart gitopiad && sudo journalctl -u gitopiad -f -o cat
```

### Create validator
```python
gitopiad tx staking create-validator \
  --amount 1000000utlore \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(gitopiad tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id gitopia-janus-testnet-2 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```python
sudo systemctl stop gitopiad && \
sudo systemctl disable gitopiad && \
rm /etc/systemd/system/gitopiad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf gitopia && \
rm -rf .gitopia && \
rm -rf $(which gitopiad)
```
#
### Sync Info
```python
gitopiad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
gitopiad status 2>&1 | jq .NodeInfo
```
### Check node logs
```bash
sudo journalctl -u gitopiad -f -o cat
```
### Check Balance
```bash
gitopiad query bank balances gitopia...address1yjgn7z09ua9vms259j
```
