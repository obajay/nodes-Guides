# Bitsong Mainnet guide

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

# Build

    git clone https://github.com/bitsongofficial/go-bitsong/
    cd go-bitsong
    git checkout v0.10.0
    make install

    bitsongd init <moniker-name> --chain-id bitsong-2b

## Create/recover wallet

    bitsongd keys add <walletname>
    bitsongd keys add <walletname> --recover

#### when creating, do not forget to write down the seed phrase
## Genesis

    wget -O $HOME/.bitsongd/config/genesis.json "https://raw.githubusercontent.com/bitsongofficial/networks/master/bitsong-2b/genesis.json"


## Set up the minimum gas price $HOME/.bitsongd/config/app.toml as well as seed and peers

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001btsg\"/;" ~/.bitsongd/config/app.toml

    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.bitsongd/config/config.toml

    peers="2af83a42fc643ddeeb5a1a10afc70fe0fa7e9201@wisdom.bonded.zone:26956"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bitsongd/config/config.toml
    seeds=""
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.bitsongd/config/config.toml 

 [SnapShot](https://sync.bonded.zone/mainnets/bitsong)

### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.bitsongd/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.bitsongd/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.bitsongd/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.bitsongd/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.bitsongd/config/config.toml

# Create a service file

    sudo tee /etc/systemd/system/bitsongd.service > /dev/null <<EOF
    [Unit]
    Description=bitsong
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which bitsongd) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

    Snapshot (optional)
## Start

    sudo systemctl daemon-reload
    sudo systemctl enable bitsongd
    sudo systemctl restart bitsongd
    sudo journalctl -u bitsongd -f -o cat

## Create validator


    bitsongd tx staking create-validator \
    --amount 1000000ubtsg \
    --from <walletName> \
    --commission-max-change-rate "0.01" \
    --commission-max-rate "0.2" \
    --commission-rate "0.05" \
    --min-self-delegation "1" \
    --details="" \
    --identity="" \
    --pubkey  $(bitsongd tendermint show-validator) \
    --moniker <moniker> \
    --fees 2000ubtsg \
    --chain-id bitsong-2b -y

## Delete node

    sudo systemctl stop bitsongd && \
    sudo systemctl disable bitsongd && \
    rm /etc/systemd/system/bitsongd.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .bitsongd && \
    rm -rf go-bitsong && \
    rm -rf $(which bitsongd)

