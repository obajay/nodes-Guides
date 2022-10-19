# Uptick testnet guide
![Uptick](https://user-images.githubusercontent.com/44331529/180614523-9a7e76e9-9243-4f38-8938-1cdaa13e2cf6.png)

[Website](https://uptick.network/ ) \
[EXPLORER](https://explorer.testnet.uptick.network/uptick-network-testnet/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   2| 4GB  | 100GB    |

### Preparing the server

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## GO 18.3 (one command)

    ver="1.18.3" && \
    cd $HOME && \
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
    sudo rm -rf /usr/local/go && \
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
    rm "go$ver.linux-amd64.tar.gz" && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
    source $HOME/.bash_profile && \
    go version

# Build 03.07.22

    git clone https://github.com/UptickNetwork/uptick.git && cd uptick
    git checkout v0.2.0
    make install
    uptickd version
      *0.2.0*
    
    uptickd init <moniker> --chain-id uptick_7776-1
    uptickd config chain-id uptick_7776-1

## Create/recover wallet

    uptickd keys add <walletname>
    uptickd keys add <walletname> --recover

## Genesis

    wget -O $HOME/.uptickd/config/genesis.json "https://raw.githubusercontent.com/UptickNetwork/uptick-testnet/main/uptick_7776-1/genesis.json"

## Peers/Seeds/Gas
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0auptick\"/;" ~/.uptickd/config/app.toml
    external_address=$(wget -qO- eth0.me)
    peers="f046ee3ead7e709b0fd6d5b30898e96959c1144d@peer0.testnet.uptick.network:26656,02ee3a0f3a2002d11c5eeb7aa813b64c59d6b60e@peer1.testnet.uptick.network:26656,51c2c58bba454c2fc7dcd6f6c32125c6b1ef3f87@161.97.130.125:26656"
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.uptickd/config/config.toml
    seeds="7aad751eb956d65388f0cc37ab2ea179e2143e41@seed0.testnet.uptick.network:26656,7e6c759bcf03641c65659f1b9b2f05ec9de7391b@seed1.testnet.uptick.network:26656"
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.uptickd/config/config.toml

### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.uptickd/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.uptickd/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.uptickd/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.uptickd/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.uptickd/config/config.toml

## State Sync
    sudo systemctl stop uptickd

    SNAP_RPC="https://uptick-testnet.nodejumper.io:443"

    LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
    BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
    TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

    echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

    peers="ce7e61b565292d6606fc0fbf4b2bc364227a1ef0@uptick-testnet.nodejumper.io:30656"
    sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.uptickd/config/config.toml

    sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.uptickd/config/config.toml

    uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book
    *sudo systemctl restart uptickd && journalctl -u uptickd -f -o cat*  (if node running)


# Create a service file

    sudo tee /etc/systemd/system/uptickd.service > /dev/null <<EOF
    [Unit]
    Description=uptick
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which uptickd) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

## Start

    sudo systemctl daemon-reload && \
    sudo systemctl enable uptickd && \
    sudo systemctl restart uptickd && sudo journalctl -u uptickd -f -o cat

## Create validator
    
    uptickd tx staking create-validator \
    --chain-id uptick_7776-1 \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.1 \
    --min-self-delegation="1000000" \
    --amount=1000000auptick \
    --pubkey $(uptickd tendermint show-validator) \
    --moniker "<name_moniker>" \
    --from=<name_wallet> \
    --gas="auto" \
    --fees 555auptick


## Delete node

    sudo systemctl stop uptickd && \
    sudo systemctl disable uptickd && \
    rm /etc/systemd/system/uptickd.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .uptickd && \
    rm -rf uptick && \
    rm -rf $(which uptickd)

