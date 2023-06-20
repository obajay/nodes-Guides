# Ollo testnet guide

![ollo](https://user-images.githubusercontent.com/44331529/192701380-3b4042b5-c257-4c25-b586-c806aa994761.png)


[EXPLORER](http://explorer.stavr.tech/ollo/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 160GB    |


# 1) Auto_install script 
```bash
wget -O olo https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Ollo/olo && chmod +x olo && ./olo
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

# Build 05.11.22
```bash
cd $HOME
git clone https://github.com/OllO-Station/ollo.git
cd ollo
git checkout v0.0.1
make install
```

`ollod version`
- latest

```bash
ollod init STAVRguide --chain-id ollo-testnet-1
ollod config keyring-backend test
```    

## Create/recover wallet
```bash
ollod keys add <walletname>
ollod keys add <walletname> --recover
```

## Download Genesis

```bash
wget -O ~/.ollo/config/genesis.json https://raw.githubusercontent.com/OllO-Station/networks/master/ollo-testnet-1/genesis.json
```
`sha256sum $HOME/.ollo/config/genesis.json`
+ 4852e73a212318cabaa6bf264e18e8aeeb42ee1e428addc0855341fad5dc7dae

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utollo\"/" $HOME/.ollo/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.ollo/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.ollo/config/config.toml
peers="a99fc4e81770ca32d574cac2e8680dccc9b55f74@18.144.61.148:26656,70ba32724461c7ed4ec8d6ddc8b5e0b1cfb9e237@54.219.57.63:26656,7864a2e4b42e5af76a83a8b644b9172fa1e40fa5@52.8.174.235:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.ollo/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.ollo/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.ollo/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.ollo/config/config.toml

```
### Pruning (optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.ollo/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.ollo/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.ollo/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.ollo/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.ollo/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.ollo/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Ollo/addrbook.json"
```

# StateSync
```bash
peers="https://ollo.rpc.bccnodes.com:443" 
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.ollo/config/config.toml
SNAP_RPC="https://ollo.rpc.bccnodes.com:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.ollo/config/config.toml
ollod tendermint unsafe-reset-all --home $HOME/.ollo
sudo systemctl restart ollod && journalctl -u ollod -f -o cat

```

# Create a service file
```bash
sudo tee /etc/systemd/system/ollod.service > /dev/null <<EOF
[Unit]
Description=ollo
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ollod) start
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
sudo systemctl enable ollod
sudo systemctl restart ollod && sudo journalctl -u ollod -f -o cat
```

### Create validator
```bash
ollod tx staking create-validator \
--amount=10000000utollo \
--pubkey=$(ollod tendermint show-validator) \
--moniker=STAVRguide \
--chain-id=ollo-testnet-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--from=<walletname> \
--identity="" \
--details="" \
--website="" \
-y

```

## Delete node
```bash
sudo systemctl stop ollod && \
sudo systemctl disable ollod && \
rm /etc/systemd/system/ollod.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf ollo && \
rm -rf .ollo && \
rm -rf $(which ollod)
```

