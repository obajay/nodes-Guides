
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
| Testnet   |   4|  8GB | 250GB    |


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

# Build 15.01.23
```python
git clone https://github.com/defund-labs/defund
cd defund
git checkout v0.2.2
make install
```
`defundd version`
- version: v0.2.2

```python
defundd init STAVRguide --chain-id defund-private-4
defundd config chain-id defund-private-4
```    

## Create/recover wallet
```python
defundd keys add <walletname>
defundd keys add <walletname> --recover
```

## Download Genesis
```python
cd $HOME/.defund/config
curl -s https://raw.githubusercontent.com/defund-labs/testnet/main/defund-private-4/genesis.json > ~/.defund/config/genesis.json
```
`sha256sum $HOME/.defund/config/genesis.json`
+ db13a33fbb4048c8701294de79a42a2b5dff599d653c0ee110390783c833208b

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ufetf\"/;" ~/.defund/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.defund/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.defund/config/config.toml
peers="d837b7f78c03899d8964351fb95c78e84128dff6@174.83.6.129:30791,f03f3a18bae28f2099648b1c8b1eadf3323cf741@162.55.211.136:26656,f8fa20444c3c56a2d3b4fdc57b3fd059f7ae3127@148.251.43.226:56656,70a1f41dea262730e7ab027bcf8bd2616160a9a9@142.132.202.86:17000,e47e5e7ae537147a23995117ea8b2d4c2a408dcb@172.104.159.69:45656,74e6425e7ec76e6eaef92643b6181c42d5b8a3b8@defund-testnet-seed.itrocket.net:443"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.defund/config/config.toml
seeds=""
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
SNAP_RPC=https://t-defund.rpc.utsa.tech:443
peers="https://t-defund.rpc.utsa.tech:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.defund/config/config.toml
defundd tendermint unsafe-reset-all --home $HOME/.defund
systemctl restart defundd && journalctl -u defundd -f -o cat
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
  --chain-id defund-private-4 \
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

