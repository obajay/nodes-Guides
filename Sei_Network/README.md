# Guide install node - Sei_Network



## Testnet details

Network Chain ID: atlantic-1 \
Denom: usei \
official instruction: https://docs.seinetwork.io/nodes-and-validators/joining-testnets \
Explorer: https://sei.explorers.guru/validators 

## Server preparation
### Updating the repositories

      sudo apt update && sudo apt upgrade -y

### Installing the necessary utilities 

      sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y


### Build GO (one command)

     ver="1.18.1" && \
     wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
     sudo rm -rf /usr/local/go && \
     sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
     rm "go$ver.linux-amd64.tar.gz" && \
     echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
     source $HOME/.bash_profile && \
     go version

## Node installation 
### IMPORTANT - currently 1.0.7beta needs to be loaded using snapshot or statesync

### Installing the binaries (22.07.22)
    
    git clone https://github.com/sei-protocol/sei-chain.git
    cd sei-chain
    git checkout master && git pull
    git checkout 1.0.7beta-postfix
    make install
    seid version --long | head
		*version: 1.0.7beta-postfix*
		*commit: 6a8d8798c75fd9e2136e599ecc97dd541b0474b4*

    
## Initializing the node to create the necessary configuration files
    seid init <name_moniker> --chain-id atlantic-1

## Downloading Genesis
    wget -O $HOME/.sei/config/genesis.json "https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/genesis.json"

### Let's check the genesis
    sha256sum ~/.sei/config/genesis.json
#### 4ae7193446b53d78bb77cab1693a6ddf6c1fe58c9693ed151e71f43956fdb3f7

## Set up node configuration
### set the minimum price for gas in app.toml
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0usei\"/;" ~/.sei/config/app.toml

### add seeds/peers to config.toml
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sei/config/config.toml
    peers="a37d65086e78865929ccb7388146fb93664223f7@18.144.13.149:26656,8ff4bd654d7b892f33af5a30ada7d8239d6f467b@91.223.3.190:51656,c4e8c9b1005fe6459a922f232dd9988f93c71222@65.108.227.133:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sei/config/config.toml
    seeds=""
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sei/config/config.toml

## (OPTIONAL) Set up pruning with one command in app.toml
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sei/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sei/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sei/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sei/config/app.toml

## (OPTIONAL) Turn off indexing in config.toml
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sei/config/config.toml

## Create a service file

    sudo tee /etc/systemd/system/seid.service > /dev/null <<EOF
    [Unit]
    Description=seid
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
    
# State Sync

SNAP_RPC="https://sei-testnet.nodejumper.io:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="4b5fb7390e9c64bc96f048816f472f4559fafd94@sei-testnet.nodejumper.io:28656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.sei/config/config.toml

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.sei/config/config.toml

    
## START
    sudo systemctl daemon-reload && \
    sudo systemctl enable seid && \
    sudo systemctl restart seid && sudo journalctl -u seid -f -o cat


## create wallet or restore wallet
    seid keys add <name_wallet>
	  or
    seid keys add <name_wallet> --recover

# Don't forget to save the seed !!!

We take test coins in discord

## Create a validator
    seid tx staking create-validator \
    --chain-id atlantic-1 \
    --commission-rate 0.05 \
    --commission-max-rate 0.2 \
    --commission-max-change-rate 0.1 \
    --min-self-delegation 1 \
    --amount 1000000usei \
    --pubkey $(seid tendermint show-validator) \
    --moniker "<name_moniker>" \
    --from <name_wallet> \
    --fees 5550usei
    
