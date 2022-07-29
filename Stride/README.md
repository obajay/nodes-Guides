# Guide Stride 
![stride](https://user-images.githubusercontent.com/44331529/180614293-57dff376-2d34-4480-803a-e8262bf37fdd.png)

[Website](https://stride.zone/)
=
[EXPLORER](https://poolparty.stride.zone/STRIDE/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   2| 4GB  | 100GB    |
### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y

## GO 18.1 (once command)
    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

### If the old network (STRIDE-1) was installed - remove the past components and continue according to the instructions (backup your priv_validator_key.json befor delete to restore your past validator)
    sudo systemctl stop strided && \
    sudo systemctl disable strided && \
    rm /etc/systemd/system/strided.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .stride && \
    rm -rf stride && \
    rm -rf $(which strided)

# Binary   27.07.22
    git clone https://github.com/Stride-Labs/stride.git
    cd stride
    git checkout 644c7574ee79128970a81cf8b9f23351dcdeec62
    sh ./scripts-local/build.sh -s $HOME/go/bin
`strided version --long | head`



## Initialisation
    strided init <moniker> --chain-id STRIDE-TESTNET-2

## Add wallet
    strided keys add <walletName>
    strided keys add <walletName> --recover

# Genesis
    wget -O $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json"

`sha256sum $HOME/.stride/config/genesis.json`
- ea1c42e096d6f3188929c68d51ad0aa514964d82b61ebb964413ddbda35b23c7  genesis.json

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

### Set up the minimum gas price and Peers/Seeds
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ustrd\"/;" ~/.stride/config/app.toml

    external_address=$(wget -qO- eth0.me) 
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.stride/config/config.toml

    peers="b61ea4c2c549e24c1a4d2d539b4d569d2ff7dd7b@stride-node1.poolparty.stridenet.co:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stride/config/config.toml

    seeds="c0b278cbfb15674e1949e7e5ae51627cb2a2d0a9@seedv2.poolparty.stridenet.co:26656"
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.stride/config/config.toml

    sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.stride/config/config.toml
    sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.stride/config/config.toml

## Download addrbook
    wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Stride/addrbook.json"


# Create a service file
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

## State Sync
    strided tendermint unsafe-reset-all --home $HOME/.stride
    peers="73f15ad99a0ac6e60cda2b691bc5b71cd7f221bc@141.95.124.151:20086"

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
    

# Start node (one command)
    sudo systemctl daemon-reload && \
    sudo systemctl enable strided && \
    sudo systemctl restart strided && \
    sudo journalctl -u strided -f -o cat

## Create validator
    strided tx staking create-validator \
    --amount=1000000ustrd \
    --pubkey=$(strided tendermint show-validator) \
    --moniker=<moniker> \
    --chain-id=STRIDE-TESTNET-2 \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.1" \
    --min-self-delegation="1" \
    --fees=100ustrd \
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
