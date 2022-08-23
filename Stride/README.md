# Guide Stride 
![stride](https://user-images.githubusercontent.com/44331529/180614293-57dff376-2d34-4480-803a-e8262bf37fdd.png)

[Website](https://stride.zone/)
=
[EXPLORER 1](https://poolparty.stride.zone/STRIDE/staking) \
[EXPLORER 2](https://stride.explorers.guru/)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4| 16GB  | 200GB    |

# 1) Auto_install script 
```bash
wget -O stride-x https://raw.githubusercontent.com/obajay/nodes-Guides/main/Stride/stride-x && chmod +x stride-x && ./stride-x
```
# 2) Manual installation

### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y

## GO 18.3 (one command)
```
ver="1.18.3" && \
cd $HOME && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
# Binary   19.08.22
```console 
git clone https://github.com/Stride-Labs/stride.git && cd stride
git checkout cf4e7f2d4ffe2002997428dbb1c530614b85df1b
make build
mkdir -p $HOME/go/bin
sudo mv build/strided /root/go/bin/
```
`strided version --long | head`
+ version: v0.3.1


## Initialisation
```console
strided init <moniker> --chain-id STRIDE-TESTNET-4
```
## Add wallet
```console
strided keys add <walletName>
strided keys add <walletName> --recover
```
# Genesis
```console
wget -O $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json"
```

`sha256sum $HOME/.stride/config/genesis.json`
- a1f56de30c4f88de2fe2fbff1a019583bfc57e9c2c297294ce2c7ec243e46a4e  genesis.json

### Pruning (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stride/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stride/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stride/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stride/config/app.toml

### Indexer (optional)
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stride/config/config.toml

### Set up the minimum gas price and Peers/Seeds/Filter peers
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ustrd\"/;" ~/.stride/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.stride/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.stride/config/config.toml

peers="54a11c47658ebd5dcbd70eb3c62197b439482d3f@116.202.236.115:21016"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stride/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.stride/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.stride/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.stride/config/config.toml
```

## Download addrbook
```console
wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Stride/addrbook.json"
```

# Create a service file
```console
sudo tee /etc/systemd/system/strided.service > /dev/null <<EOF
[Unit]
Description=strided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which strided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync STRIDE
```bash
peers="cf4e629008be3417081395f4ab6f591a74124d90@141.95.124.151:20086"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stride/config/config.toml

SNAP_RPC=141.95.124.151:20087
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.stride/config/config.toml
strided tendermint unsafe-reset-all --home $HOME/.stride
sudo systemctl restart strided && journalctl -u strided -f -o cat
```
# StateSync GAIA
```console
sudo systemctl stop gaiad
PEERS="f42f1908713b97a2106a0872c7a6dbf64b274a77@141.95.124.151:23656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.gaia/config/config.toml
SNAP_RPC="141.95.124.151:23657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.gaia/config/config.toml
gaiad tendermint unsafe-reset-all --home $HOME/.gaia
sudo systemctl restart gaiad && journalctl -u gaiad -f -o cat
```

# Start node (one command)
```console
sudo systemctl daemon-reload && \
sudo systemctl enable strided && \
sudo systemctl restart strided && \
sudo journalctl -u strided -f -o cat
```

## Create validator
    strided tx staking create-validator \
    --amount=1000000ustrd \
    --pubkey=$(strided tendermint show-validator) \
    --moniker=<moniker> \
    --chain-id=STRIDE-TESTNET-4 \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.1" \
    --min-self-delegation="1" \
    --fees=500ustrd \
    --from=<walletName> \
    --identity="" \
    --website="" \
    --details="" \
    -y


### Delete node (one command)
    sudo systemctl stop strided && \
    sudo systemctl disable strided && \
    rm /etc/systemd/system/strided.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .stride && \
    rm -rf stride && \
    rm -rf $(which strided)
