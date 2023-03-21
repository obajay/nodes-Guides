
# DeFund Testnet guide

![defund](https://user-images.githubusercontent.com/44331529/198265844-d3d66bd2-78cf-4c14-8320-12a00f2aafe0.png)

[WEBSITE](https://defund.app/) \
[GitHub](https://github.com/defund-labs/testnet)
=
[EXPLORER 1](https://explorer.stavr.tech/defund-testnet/staking) \
[EXPLORER 2](https://defund.explorers.guru/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Tetsnet   |   4|  8GB | 250GB    |


# 1) Auto_install script
```python
wget -O dfn https://raw.githubusercontent.com/obajay/nodes-Guides/main/DeFund/dfn && chmod +x dfn && ./dfn
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

# Build 20.03.23
```python
git clone https://github.com/defund-labs/defund
cd defund
git checkout v0.2.6
make install
```
*******🟢UPDATE🟢******* 20.03.23

```python
cd $HOME/defund
git fetch --all
git checkout v0.2.6
make install
defundd version --long
 #version: v0.2.6
 #commit: 8f2ebe3d30efe84e013ec5fcdf21a3b99e786c3d
sudo systemctl restart defundd && journalctl -u defundd -f -o cat
```

`defundd version --long`
- version: v0.2.6
- commit: 8f2ebe3d30efe84e013ec5fcdf21a3b99e786c3d

```python
defundd init STAVRguide --chain-id orbit-alpha-1
defundd config chain-id orbit-alpha-1
```    

## Create/recover wallet
```python
defundd keys add <walletname>
defundd keys add <walletname> --recover
```

## Download Genesis
```python
cd $HOME/.defund/config
curl -s https://raw.githubusercontent.com/defund-labs/testnet/main/orbit-alpha-1/genesis.json > ~/.defund/config/genesis.json
```
`sha256sum $HOME/.defund/config/genesis.json`
+ 58916f9c7c4c4b381f55b6274bce9b8b8d482bfb15362099814ff7d0c1496658

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ufetf\"/;" ~/.defund/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.defund/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.defund/config/config.toml
peers="f902d7562b7687000334369c491654e176afd26d@170.187.157.19:26656,f8093378e2e5e8fc313f9285e96e70a11e4b58d5@rpc-2.defund.nodes.guru:45656,878c7b70a38f041d49928dc02418619f85eecbf6@rpc-3.defund.nodes.guru:45656,3594b1f46c6321d9f99cda8ad5ef5a367ce06ccf@199.247.16.116:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.defund/config/config.toml
seeds="f902d7562b7687000334369c491654e176afd26d@170.187.157.19:26656,2b76e96658f5e5a5130bc96d63f016073579b72d@rpc-1.defund.nodes.guru:45656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.defund/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.defund/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.defund/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.defund/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.defund/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.defund/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/DeFund/addrbook.json"
```

## StateSync
```python
SOOON
```

# Create a service file
```python
sudo tee /etc/systemd/system/defundd.service > /dev/null <<EOF
[Unit]
Description=defund
After=network-online.target

[Service]
User=$USER
ExecStart=$(which defundd) start
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
sudo systemctl enable defundd
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
```

### Create validator
```python
defundd tx staking create-validator \
  --amount 1000000ufetf \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(defundd tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id orbit-alpha-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```python
sudo systemctl stop defundd && \
sudo systemctl disable defundd && \
rm /etc/systemd/system/defundd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf defund && \
rm -rf .defund && \
rm -rf $(which defundd)
```

#
### Sync Info
```python
defundd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
defundd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
defundd journalctl -u haqqd -f -o cat
```
### Check Balance
```python
defundd query bank balances defund...addressdefund1yjgn7z09ua9vms259j
```

