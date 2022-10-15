# Lambda Mainnet guide


[<img src='https://user-images.githubusercontent.com/44331529/195008212-3489c979-2416-4df7-bedc-fc351956ea85.png'>](https://explorer.lambda.im/)

[Website](https://lambda.im/)
=
[EXPLORER 1](https://explorer.stavr.tech/lambda/staking) \
[EXPLORER 2](https://explorer.nodestake.top/lambda/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 160GB    |


# 1) Auto_install script
```bash
SOON
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

# Build 10.10.22
```bash
cd ~
git clone https://github.com/LambdaIM/lambdavm.git
cd lambdavm
make install
```
`lambdavm version`
- 1.0.0

```bash
lambdavm init STAVRguide --chain-id lambda_92000-1
```    

## Create/recover wallet
```bash
lambdavm keys add <walletname>
lambdavm keys add <walletname> --recover
```

## Download Genesis

```bash
wget https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/genesis.json
mv genesis.json ~/.lambdavm/config/
```
`sha256sum $HOME/.lambdavm/config/genesis.json`
+ 1ff02001539bc1e9828fe170006f055c04df280c61c4ca9ecc9e7b6a272b7777

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ulamb\"/" $HOME/.lambdavm/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.lambdavm/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.lambdavm/config/config.toml
peers="2c4f8e193fd10ab3a2bc919b484fd1c78eceffb3@13.213.214.88:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lambdavm/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.lambdavm/config/config.toml
sed -i -e "s/^timeout_commit *=.*/timeout_commit = \"2s\"/" $HOME/.lambdavm/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.lambdavm/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.lambdavm/config/config.toml

```
### Pruning (optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.lambdavm/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.lambdavm/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.lambdavm/config/addrbook.json "soon"
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
sudo tee /etc/systemd/system/lambdavm.service > /dev/null <<EOF
[Unit]
Description=lambdavm
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lambdavm) start
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
sudo systemctl enable lambdavm
sudo systemctl restart lambdavm && sudo journalctl -u lambdavm -f -o cat
```

### Create validator
```bash
lambdavm tx staking create-validator \
  --amount="1000000000000000000"ulamb \
  --pubkey=$(lambdavm tendermint show-validator) \
  --moniker=STAVRguide \
  --chain-id=lambda_92000-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.1" \
  --min-self-delegation="1" \
  --from=<walletName> \
  --fees 3500ulamb \
  --gas 350000 \
  --identity="" \
  --details="" \
  --website="" \
  -y
```

## Delete node
```bash
sudo systemctl stop lambdavm && \
sudo systemctl disable lambdavm && \
rm /etc/systemd/system/lambdavm.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf lambdavm && \
rm -rf .lambdavm && \
rm -rf $(which lambdavm)
```
