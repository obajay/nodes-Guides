# Nois testnet guide V003

![nnnoi](https://user-images.githubusercontent.com/44331529/191945004-1227fef0-a215-44f1-bcab-854acd66de00.png)

[Website](https://nois.network/)
=
[EXPLORER 1](http://explorer.stavr.tech/nois/staking) \
[EXPLORER 2](https://testnet.ping.pub/nois/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   8| 16GB | 160GB    |


# 1) Auto_install script
```bash
wget -O nois https://raw.githubusercontent.com/obajay/nodes-Guides/main/Nois/nois && chmod +x nois && ./nois
```

# 2) Manual installation

### Preparing the server

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 18.5 

```bash
cd $HOME
ver="1.18.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 07.10.22 
```bash
cd ~
git clone https://github.com/noislabs/full-node.git
cd full-node/full-node/
git checkout nois-testnet-003
./build.sh
mv out/noisd /usr/local/bin
```
`noisd version`
- 0.29.0-rc2

```bash
noisd init STAVRguide --chain-id nois-testnet-003
```    

## Create/recover wallet
```bash
noisd keys add <walletname>
noisd keys add <walletname> --recover
```

## Download Genesis

```bash
cd $HOME/.noisd/config/
rm genesis.json
curl -O https://raw.githubusercontent.com/noislabs/testnets/main/nois-testnet-003/genesis.json
```
`sha256sum $HOME/.noisd/config/genesis.json`
+ 9153084f305111e72fed86f44f6a11711c421532722200c870170d98223233ba

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.05unois\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.noisd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.noisd/config/config.toml
peers="2bf8002d0f65c3d86fca31ea0f043d912682c3e0@65.109.70.23:17356"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.noisd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.noisd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.noisd/config/config.toml

```
### Pruning (optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.noisd/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.noisd/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.noisd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Nois/addrbook.json"
```

# StateSync
```bash
SNAP_RPC=https://nois-testnet-rpc.polkachu.com:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.noisd/config/config.toml
noisd tendermint unsafe-reset-all --home $HOME/.noisd --keep-addr-book
systemctl restart noisd && journalctl -u noisd -f -o cat
```

# Create a service file
```bash
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=noisd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noisd) start --home $HOME/.noisd
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
sudo systemctl enable noisd
sudo systemctl restart noisd && sudo journalctl -u noisd -f -o cat
```

### Create validator
```bash
noisd tx staking create-validator \
--amount=99000000unois \
--pubkey=$(noisd tendermint show-validator) \
--moniker=STAVRguide \
--chain-id=nois-testnet-003 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--from=<wallet> \
--fees=16000unois \
--gas=300000 \
-y
```

## Delete node
```bash
sudo systemctl stop noisd && \
sudo systemctl disable noisd && \
rm /etc/systemd/system/noisd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf full-node && \
rm -rf .noisd && \
rm -rf $(which noisd)
```

