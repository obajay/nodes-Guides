# Quicksilver  Testnet

![quick](https://user-images.githubusercontent.com/44331529/201520331-711f381d-89ab-4b8b-bab9-114c2b2521bd.png)


[WEBSITE](https://quicksilver.zone/)
=
[EXPLORER 1](https://explorer.stavr.tech/quicksilver/staking) \
[EXPLORER 2](https://testnet.manticore.team/quicksilver/staking)
=
### Updating the repositories  

    sudo apt update && sudo apt upgrade -y

### Installing the necessary utilities

    sudo apt install curl build-essential git wget jq make gcc tmux nvme-cli -y
    
### GO 1.18.1 (one command)

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

### Node installation 28.11.22
```python
cd $HOME
wget https://github.com/ingenuity-build/testnets/releases/download/v0.10.0/quicksilverd-v0.10.3-amd64
mv quicksilverd-v0.10.3-amd64 quicksilverd
chmod +x quicksilverd
mv $HOME/quicksilverd $HOME/go/bin/
```
`quicksilverd version`
+ version: v0.10.3


### Initialize the node
```java
quicksilverd init STAVRguide --chain-id innuendo-3
```

### Create wallet or restore
    quicksilverd keys add <name_wallet>
            or
    quicksilverd keys add <name_wallet> --recover

### Download Genesis
```python
wget -O $HOME/.quicksilverd/config/genesis.json "https://raw.githubusercontent.com/ingenuity-build/testnets/main/innuendo/genesis.json"
```
`sha256sum ~/.quicksilverd/config/genesis.json`
 + 6f97a06cdcfddc5774d4ca4fbee936bc8462b72b74c4337753771fecdfebe93f

### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uqck\"/;" ~/.quicksilverd/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.quicksilverd/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.quicksilverd/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.quicksilverd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.quicksilverd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.quicksilverd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.quicksilverd/config/config.toml
```


### Setting up pruning with one command (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="50" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.quicksilverd/config/app.toml

### Disable indexing (optional)
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.quicksilverd/config/config.toml

### StateSync
```python
SNAP_RPC=https://quicksilver-t.rpc.manticore.team:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" ~/.quicksilverd/config/config.toml
quicksilverd tendermint unsafe-reset-all --home /root/.quicksilverd
systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

### Download addrbook
```python
wget -O $HOME/.quicksilverd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quicksilver/addrbook.json"
```

### Create a service file
```python
sudo tee /etc/systemd/system/quicksilverd.service > /dev/null <<EOF
[Unit]
Description=quicksilver
After=network-online.target

[Service]
User=$USER
ExecStart=$(which quicksilverd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```python    
sudo systemctl daemon-reload && \
sudo systemctl enable quicksilverd && \
sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

#### After synchronization, we go to the discord and we request coins

### Create a validator
```python
quicksilverd tx staking create-validator \
--chain-id innuendo-3 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.1 \
--min-self-delegation="1" \
--amount=1000000uqck \
--pubkey $(quicksilverd tendermint show-validator) \
--moniker "STAVRguide" \
--from=<name_wallet> \
--gas="auto" \
--fees 555uqck -y
```    

#

`Sync Info`
```python
quicksilverd status 2>&1 | jq .SyncInfo
```
`NodeINfo`
```python
quicksilverd status 2>&1 | jq .NodeInfo
```
`Check node logs`
```python
quicksilverd journalctl -u haqqd -f -o cat
```
`Check Balance`
```python
quicksilverd query bank balances quicksilver...addressdefund1yjgn7z09ua9vms259j
```
