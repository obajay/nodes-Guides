[EXPLORER](https://testnet.owlstake.com/Aura-Network/staking)
# Aura testnet guide

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## GO 18.1 (one command)

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

# Build 11.07.22

    git clone https://github.com/aura-nw/aura
    cd aura
    git checkout euphoria
    make install
    aurad version --long | head
    euphoria_4027003
    
    aurad init <moniker> --chain-id euphoria-1

## Create/recover wallet

    aurad keys add <walletname>
    aurad keys add <walletname> --recover

## Genesis

    wget -O $HOME/.aura/config/genesis.json "https://raw.githubusercontent.com/aura-nw/testnets/main/euphoria-1/genesis.json"
    sha256sum ~/.aura/config/genesis.json
    732f789aefe39e47b969f1e5ce440a675d8f6a640368d70e4d7f628b3c87e831

## Download addrbook

    wget -O $HOME/.aura/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Aura/addrbook.json"


## Minimum gas price/Peers/Seeds

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ueaura\"/;" ~/.aura/config/app.toml
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.aura/config/config.toml
    peers="007a30814c0dd48e9239bc874a7d8406dac6cd53@65.108.46.123:26156,64fdaa6da59901793beda215679ac2a6549b46b4@144.91.122.166:26656,d3ba3d354b657ec21377791d307a4ed01b2898d7@138.201.139.175:20356,c53157159e7cea010b86e44786831f792d852e1f@135.181.72.187:26656,594f32a7496097e5c8cecd23156862e714c9a729@144.76.224.246:56656,5d869eb132e188b848875cc169edb3614d6bb620@144.76.27.79:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.aura/config/config.toml
    seeds="705e3c2b2b554586976ed88bb27f68e4c4176a33@13.250.223.114:26656,b9243524f659f2ff56691a4b2919c3060b2bb824@13.214.5.1:26656"
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.aura/config/config.toml
    sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.aura/config/config.toml
    sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.aura/config/config.toml

### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.aura/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.aura/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.aura/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.aura/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.aura/config/config.toml

## State Sync
    SNAP_RPC=
    (choose 1 out of 3)
    https://snapshot-2.euphoria.aura.network:443
    https://snapshot-1.euphoria.aura.network:443
    https://rpc-t.aura.nodestake.top/

    LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
    BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
    TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

    echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

    sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
    s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.aura/config/config.toml

# Create a service file

    sudo tee /etc/systemd/system/aurad.service > /dev/null <<EOF
    [Unit]
    Description=aurad
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which aurad) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

## Start

    sudo systemctl daemon-reload && \ 
    sudo systemctl enable aurad && \
    sudo systemctl restart aurad && \
    sudo journalctl -u aurad -f -o cat

## Create validator


    aurad tx staking create-validator \
    --amount 1000000ueaura \
    --from <walletName> \
    --commission-max-change-rate "0.1" \
    --commission-max-rate "0.2" \
    --commission-rate "0.05" \
    --min-self-delegation "1" \
    --details="" \
    --identity="" \
    --pubkey  $(aurad tendermint show-validator) \
    --moniker <moniker> \
    --fees 555ueaura \
    --chain-id euphoria-1 -y


## Delete node
    sudo systemctl stop aurad && \
    sudo systemctl disable aurad && \
    rm /etc/systemd/system/aurad.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .aura && \
    rm -rf aura && \
    rm -rf $(which aurad)

