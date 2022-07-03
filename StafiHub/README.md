# StaFiHub Public Testnet v3
## Hardware Requirements
### Minimal
4GB RAM \
200GB SSD \
2 vCPU
### Recommended
8GB RAM \
300GB SSD \
4 vCPU

# Installation Steps
## Install dependencies:

    cd $HOME
    sudo apt update
    sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"

## Install Go (one command):
    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

## Clone git repository:

    git clone --branch public-testnet-v3 https://github.com/stafihub/stafihub

## Install:

    cd $HOME/stafihub && make install

## Download genesis (replace YOUR_NODE_NAME):

    stafihubd init <YOUR_NODE_NAME> --chain-id stafihub-public-testnet-3
    wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/stafihub/network/main/testnets/stafihub-public-testnet-3/genesis.json"
    stafihubd tendermint unsafe-reset-all --home ~/.stafihub

## Configure your node:

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
    sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
    peers="4e2441c0a4663141bb6b2d0ea4bc3284171994b6@46.38.241.169:26656,79ffbd983ab6d47c270444f517edd37049ae4937@23.88.114.52:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
    
### Pruning (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stafihub/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stafihub/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stafihub/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stafihub/config/app.toml

### Indexer (optional)
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stafihub/config/config.toml

## Install service to run the node:

    echo "[Unit]
    Description=StaFiHub Node
    After=network.target

    [Service]
    User=$USER
    Type=simple
    ExecStart=$(which stafihubd) start
    Restart=on-failure
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target" > $HOME/stafihubd.service
    sudo mv $HOME/stafihubd.service /etc/systemd/system
    sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
    Storage=persistent
    EOF

    sudo systemctl restart systemd-journald
    sudo systemctl daemon-reload
    sudo systemctl enable stafihubd
    sudo systemctl restart stafihubd
    
    
### Generate keys
    stafihubd keys add <YOUR_WALLET_NAME> --keyring-backend file
#### You can recover your keys with --recover flag if you have mnemonic

### Faucet
You can ask for tokens in the #faucet Discord channel after synchronization \
    !faucet send YOUR_WALLET_ADDRESS

### Create validator
#### Use the following command (do not forget to replace YOUR_NODE_NAME and YOUR_WALLET_NAME):

    stafihubd tx staking create-validator -y --amount=1000000ufis --pubkey=$(stafihubd tendermint show-validator) --moniker=<YOUR_NODE_NAME> --commission-rate=0.10 --commission-max-rate=0.20 --commission-max-change-rate=0.1 --min-self-delegation=1 --from=<YOUR_WALLET_NAME> --chain-id=stafihub-public-testnet-3 --gas-prices=0.025ufis

[Explorer](https://testnet-explorer.stafihub.io/stafi-hub-testnet/staking)


    
