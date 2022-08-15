
[EXPLORER](https://sei.explorers.guru/validators)

## UPDATE APT
    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## INSTALL GO
    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

    cd $HOME
### Installing the binaries (03.08.22)
    
    git clone https://github.com/sei-protocol/sei-chain.git
    cd sei-chain
    git checkout master && git pull
    git checkout 1.1.0beta
    make install
    seid version --long | head
+ version: 1.1.0beta
+ commit: 33e9e1d53a3fcd26748d3134d84b5748cac5e147*

    seid init <moniker> --chain-id sei-devnet-1
    seid config chain-id sei-devnet-1

### WALLET
    seid keys add <walletName> --recover

## GENESIS
    wget -O $HOME/.sei/config/genesis.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-devnet-1/genesis.json"
    wget -O $HOME/.sei/config/addrbook.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-devnet-1/addrbook.json"

### Peers and seed
    external_address=$(wget -qO- eth0.me)
    peers=""
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sei/config/config.toml
    SEEDS=""
    sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.sei/config/config.toml

### Pruning (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sei/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sei/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sei/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sei/config/app.toml

### Indexer (optional)
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sei/config/config.toml

## Create service
    sudo tee /etc/systemd/system/seid.service > /dev/null <<EOF
    [Unit]
    Description=SEI
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which seid) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF


## START WITH STATE-SYNC

    sudo systemctl stop seid
    seid tendermint unsafe-reset-all --home $HOME/.sei
    wget -O $HOME/.sei/config/addrbook.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-devnet-1/addrbook.json"
    SNAP_RPC="http://116.203.35.46:36657"
    LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
    BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
    TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

    sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
    s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.sei/config/config.toml


# start service
    sudo systemctl daemon-reload
    sudo systemctl enable seid
    sudo systemctl restart seid
    journalctl -u seid -f -o cat


## Create Validator
    seid tx staking create-validator \
      --amount 1000000usei \
      --from <walletName> \
      --commission-max-change-rate "0.05" \
      --commission-max-rate "0.20" \
      --commission-rate "0.05" \
      --min-self-delegation "1" \
      --pubkey $(seid tendermint show-validator) \
      --moniker <moniker> \
      --chain-id sei-devnet-1 \
      --gas 300000 \
      -y




