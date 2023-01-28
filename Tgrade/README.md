# Tgrade Mainnet guide

![tg](https://user-images.githubusercontent.com/44331529/196773031-a0ed0f92-5646-485d-a47f-2e9675f684c1.png)

[Tgrade Dapp](https://dapp.tgrade.finance/trustedcircle) \
[GitHub](https://github.com/mande-labs)
=
[EXPLORER](https://www.mintscan.io/tgrade)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 300GB    |


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

# Build 18.10.22
```bash
cd ~
git clone https://github.com/confio/tgrade
cd tgrade
git checkout v2.0.2
make build
sudo mv $HOME/tgrade/build/tgrade /usr/local/bin/
```
`tgrade version --long | head`
- version: 2.0.2
- commit: a990225d4b92f09d7b6fad3d6147d71c3711ece4

```bash
tgrade init STAVRguide --chain-id tgrade-mainnet-1
```    

## Create/recover wallet
```bash
tgrade keys add <walletname>
tgrade keys add <walletname> --recover
```

## Download Genesis

```bash
wget https://raw.githubusercontent.com/confio/tgrade-networks/main/mainnet-1/config/genesis.json -O $HOME/.tgrade/config/genesis.json
```
`sha256sum $HOME/.tgrade/config/genesis.json`
+ 26d4bced80fe45009c8202df3b1731c24bc0100793279b95083d5b97dc9671a1

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.05utgd\"/;" $HOME/.tgrade/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.tgrade/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.tgrade/config/config.toml
peers="0a63421f67d02e7fb823ea6d6ceb8acf758df24d@142.132.226.137:26656,4a319eead699418e974e8eed47c2de6332c3f825@167.235.255.9:26656,6918efd409684d64694cac485dbcc27dfeea4f38@49.12.240.203:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.tgrade/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.tgrade/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.tgrade/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.tgrade/config/config.toml

```
### Pruning (optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.tgrade/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.tgrade/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.tgrade/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.tgrade/config/app.toml
```

### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.tgrade/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.tgrade/config/addrbook.json "SOON"
```

# StateSync
```bash
SNAP_RPC=https://tgrade-rpc.anyvalid.com:443
peers="https://tgrade-rpc.anyvalid.com:443"
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.tgrade/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.tgrade/config/config.toml
tgrade tendermint unsafe-reset-all --home /root/.tgrade --keep-addr-book
sudo systemctl restart tgrade && sudo journalctl -u tgrade -f -o cat

```

# Create a service file
```bash
sudo tee <<EOF >/dev/null /etc/systemd/system/tgrade.service
[Unit]
Description=Tgrade daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/tgrade start
Restart=on-failure
RestartSec=3
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```bash
sudo systemctl daemon-reload
sudo systemctl enable tgrade
sudo systemctl restart tgrade && sudo journalctl -u tgrade -f -o cat
```



### Create validator
```bash
tgrade tx poe create-validator \
  --amount 1000000utgd \
  --vesting-amount 0utgd \
  --from <walletName> \
  --pubkey $(sudo tgrade tendermint show-validator) \
  --chain-id tgrade-mainnet-1 \
  --moniker "STAVRguide" \
  --fees 200000utgd \
  --gas auto \
  --gas-adjustment 1.4 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop tgrade && \
sudo systemctl disable tgrade && \
rm /etc/systemd/system/tgrade.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .tgrade && \
rm -rf tgrade && \
rm -rf $(which tgrade)
```
