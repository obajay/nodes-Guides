# Neutron Testnet guide

![neutron](https://user-images.githubusercontent.com/44331529/201018597-cccdc09f-0f52-4cf9-af71-4d42ddf53e35.png)

[WebSite](https://neutron.org/)
=
[EXPLORER](https://explorer.stavr.tech/neutron-testnet/staking)
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

## GO 1.19.4

```python
cd $HOME && \
ver="1.19.4" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 08.02.23
```python
cd $HOME
git clone https://github.com/neutron-org/neutron
cd neutron
git checkout v0.2.0
make install
```
`neutrond version --head`
- version: 0.2.0
- commit: 4ebd7ae7ab1e614e8fbd9a18a8c86d8045ae787c


```python
neutrond init STAVRguide --chain-id baryon-1
neutrond config chain-id baryon-1
```    

## Create/recover wallet
```python
neutrond keys add <walletname>
              or
neutrond keys add <walletname> --recover
```

## Download Genesis
```python
curl -s https://raw.githubusercontent.com/neutron-org/cosmos-testnets/master/replicated-security/baryon-1/baryon-1-genesis.json > ~/.neutrond/config/genesis.json
```
`sha256sum $HOME/.neutrond/config/genesis.json`
+ eab7c27701d39e96793149d6e5feb27fd4bc260eb3df62c09517f3f61adcd72c

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0untrn\"/" $HOME/.neutrond/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.neutrond/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.neutrond/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.neutrond/config/config.toml
seeds="e2c07e8e6e808fb36cca0fc580e31216772841df@p2p.baryon.ntrn.info:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.neutrond/config/config.toml
sed -i -e "s/^timeout_commit *=.*/timeout_commit = \"2s\"/" $HOME/.neutrond/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.neutrond/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.neutrond/config/config.toml
```

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.neutrond/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.neutrond/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.neutrond/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.neutrond/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.neutrond/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.neutrond/config/addrbook.json "SOON"
```

# Create a service file
```python
sudo tee /etc/systemd/system/neutrond.service > /dev/null <<EOF
[Unit]
Description=neutron
After=network-online.target

[Service]
User=$USER
ExecStart=$(which neutrond) start
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
sudo systemctl enable neutrond
sudo systemctl restart neutrond && sudo journalctl -u neutrond -f -o cat
```

### Create validator
```python
neutrond tx staking create-validator \
  --amount 1000000untrn \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(neutrond tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id baryon-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```python
sudo systemctl stop neutrond && \
sudo systemctl disable neutrond && \
rm /etc/systemd/system/neutrond.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf neutron && \
rm -rf .neutrond && \
rm -rf $(which neutrond)
```
#
### Sync Info
```python
neutrond status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
neutrond status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u neutrond -f -o cat
```
### Check Balance
```python
neutrond query bank balances neutron...address1yjgn7z09ua9vms259j
```
