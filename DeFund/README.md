
# DeFund Testnet guide

![defund](https://user-images.githubusercontent.com/44331529/198265844-d3d66bd2-78cf-4c14-8320-12a00f2aafe0.png)

[WEBSITE](https://defund.app/) \
[GitHub](https://github.com/defund-labs/testnet)
=
[EXPLORER](https://defund.explorers.guru/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Tetsnet   |   4|  8GB | 250GB    |


# 1) Auto_install script
```bash
wget -O dfn https://raw.githubusercontent.com/obajay/nodes-Guides/main/DeFund/dfn && chmod +x dfn && ./dfn
```

# 2) Manual installation

### Preparing the server

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19

```bash
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 03.11.22
```bash
git clone https://github.com/defund-labs/defund
cd defund
git checkout v0.1.0
make install
```
`defundd version`
- version: v0.1.0

```bash
defundd init STAVRguide --chain-id defund-private-3

```    

## Create/recover wallet
```bash
defundd keys add <walletname>
defundd keys add <walletname> --recover
```

## Download Genesis
```bash
cd $HOME/.defund/config/
wget -O $HOME/.defund/config/defund-private-3-gensis.tar.gz "https://raw.githubusercontent.com/defund-labs/testnet/main/defund-private-3/defund-private-3-gensis.tar.gz"
rm -rf $HOME/.defund/config/genesis.json
tar -xzvf defund-private-3-gensis.tar.gz
defundd config chain-id defund-private-3 
rm -rf defund-private-3-gensis.tar.gz
```
`sha256sum $HOME/.defund/config/genesis.json`
+ 1a10121467576ab6f633a14f82d98f0c39ab7949102a77ab6478b2b2110109e3

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ufetf\"/;" ~/.defund/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.defund/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.defund/config/config.toml
peers="dff3a67755a5832198447224196654374c7ef95d@65.21.170.3:40656,daff7b8cbcae4902c3c4542113ba521f968cc3f8@213.239.217.52:29656,445425e51dc42603cfeac805816bcdda2fb8a6a1@65.109.54.110:26631,f2985029a48319330b99767d676412383e7061bf@194.163.155.84:36656,75cccc67bc20e7e5429b80c4255ffe44ef24bc26@65.109.85.170:33656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.defund/config/config.toml
seeds="85279852bd306c385402185e0125dffeed36bf22@38.146.3.194:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.defund/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.defund/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.defund/config/config.toml

```
### Pruning (optional)
```bash
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
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.defund/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.defund/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/DeFund/addrbook.json"
```

## StateSync
```bash
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
```bash
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
```bash
sudo systemctl daemon-reload
sudo systemctl enable defundd
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
```

### Create validator
```bash
defundd tx staking create-validator \
  --amount 1000000ufetf \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(defundd tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id defund-private-2 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop defundd && \
sudo systemctl disable defundd && \
rm /etc/systemd/system/defundd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf defund && \
rm -rf .defundd && \
rm -rf $(which defundd)
```

#
### Sync Info
```bash
defundd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```bash
defundd status 2>&1 | jq .NodeInfo
```
### Check node logs
```bash
defundd journalctl -u haqqd -f -o cat
```
### Check Balance
```bash
defundd query bank balances defund...addressdefund1yjgn7z09ua9vms259j
```

