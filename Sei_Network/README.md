# Guide install node - Sei_Network



## Testnet details

Network Chain ID: sei-testnet-2 \
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
### IMPORTANT - currently 1.0.4 needs to be loaded using snapshot or statesync

### Installing the binaries

    sudo systemctl restart seid && journalctl -u seid -f -o cat
    sudo systemctl stop seid && \
    sudo rm -rf $HOME/sei-chain && \
    git clone https://github.com/sei-protocol/sei-chain.git && \
    cd sei-chain && \
    git checkout 1.0.4beta && \
    make install

    seid version --long | head
### version 1.0.4beta
#### commit: 09b267bbc7d32d3f61ab1b186f59f4bc5fea8970
    sudo systemctl restart seid && journalctl -u seid -f -o cat

## Initializing the node to create the necessary configuration files
    seid init <name_moniker> --chain-id sei-testnet-2

## Downloading Genesis
    wget -O $HOME/.sei/config/genesis.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-testnet-2/genesis.json"

### Let's check the genesis
    sha256sum ~/.sei/config/genesis.json
#### aec481191276a4c5ada2c3b86ac6c8aad0cea5c4aa6440314470a2217520e2cc

## Download addrbook
    wget -O $HOME/.sei/config/addrbook.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-testnet-2/addrbook.json"

## Set up node configuration
### set the minimum price for gas in app.toml
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025usei\"/;" ~/.sei/config/app.toml

### add seeds/bpeers/peers to config.toml
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sei/config/config.toml
    peers=""
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sei/config/config.toml
    bpeers=""
    sed -i.bak -e "s/^bootstrap-peers *=.*/bootstrap-peers = \"$bpeers\"/" $HOME/.sei/config/config.toml
    seeds=""
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sei/config/config.toml

## (OPTIONAL) Set up pruning with one command in app.toml
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="50" && \
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
    --chain-id sei-testnet-2 \
    --commission-rate 0.05 \
    --commission-max-rate 0.2 \
    --commission-max-change-rate 0.1 \
    --min-self-delegation 1 \
    --amount 1000000usei \
    --pubkey $(seid tendermint show-validator) \
    --moniker "<name_moniker>" \
    --from <name_wallet> \
    --fees 5550usei
    
