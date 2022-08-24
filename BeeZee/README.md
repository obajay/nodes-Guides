# BeeZee Mainnet guide

![Beezee](https://user-images.githubusercontent.com/44331529/180596395-845e85eb-ed01-4bca-ae94-90bdbfd6e5be.png)


[Explorer](https://explorer.erialos.me/beezee/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 160GB    |

# 1) Auto_install script
```bash
wget -O beezzeed https://raw.githubusercontent.com/obajay/nodes-Guides/main/BeeZee/beezzeed && chmod +x beezzeed && ./beezzeed
```
# 2) Manual installation

### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## GO 18.1 (one command)

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

## Build
```bash
git clone https://github.com/bze-alphateam/bze
cd bze
git checkout v5.0.1
make install
```
`bzed version`
+ 5.0.1
```bash
bzed init <moniker> --chain-id beezee-1
bzed config chain-id beezee-1
```    
## Create/recover wallet

    bzed keys add <walletname>
    bzed keys add <walletname> --recover

### when creating, do not forget to write down the seed phrase

# Genesis

    wget https://raw.githubusercontent.com/bze-alphateam/bze/main/genesis.json -O $HOME/.bze/config/genesis.json

## Seeds and peers
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.bze/config/config.toml

    seeds="6385d5fb198e3a793498019bb8917973325e5eb7@51.15.228.169:26656,ce25088267cef31f3be1ec03263524764c5c80bb@163.172.130.162:26656,102d28592757192ccf709e7fbb08e7dd8721feb1@51.15.138.216:26656,f238198a75e886a21cd0522b6b06aa019b9e182e@51.15.55.142:26656,2624d40b8861415e004d4532bb7d8d90dd0e6e66@51.15.115.192:26656,d36f2bc75b0e7c28f6cd3cbd5bd50dc7ed8a0d11@38.242.227.150:26656"
    sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.bze/config/config.toml

## Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.bze/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.bze/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.bze/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.bze/config/app.toml

## Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.bze/config/config.toml

## Download addrbook

wget -O $HOME/.bze/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/BeeZee/addrbook.json"


# Create a service file

    sudo tee /etc/systemd/system/bzed.service > /dev/null <<EOF
    [Unit]
    Description=BeeZee mainnet
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which bzed) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF


## State sync (optional)
    a9fac0534bd6853f5810fdc692564967bd01b1fe@rpc-1.getbze.com:26656
    peers="a9fac0534bd6853f5810fdc692564967bd01b1fe@rpc-1.getbze.com:26656"
    sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.bze/config/config.toml

    SNAP_RPC=https://rpc-2.getbze.com:443

    LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
    BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
    TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

    echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

    sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
    s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.bze/config/config.toml
    bzed unsafe-reset-all --keep-addr-book
        
# Start

    sudo systemctl daemon-reload
    sudo systemctl enable bzed
    sudo systemctl restart bzed
    sudo journalctl -u bzed -f -o cat

## Create validator
    bzed tx staking create-validator \
    --amount=1000000ubze \
    --pubkey=$(bzed tendermint show-validator) \
    --moniker="<moniker>" \
    --identity="" \
    --details="" \
    --website="" \
    --chain-id="beezee-1" \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --min-self-delegation="1" \
    --fees 500ubze \
    --from=<walletname> -y


# Delete node

    sudo systemctl stop bzed && \
    sudo systemctl disable bzed && \
    rm /etc/systemd/system/bzed.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .bze && \
    rm -rf bze && \
    rm -rf $(which bzed)
