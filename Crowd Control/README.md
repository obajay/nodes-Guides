# Crowd Control testnet guide

![Crowd Control](https://user-images.githubusercontent.com/44331529/180597315-e25b1929-8973-4149-b2c6-b9086c1787bd.png)

[EXPLORER 1](https://explorer.theamsolutions.info/Cardchain/staking)
=
[EXPLORER 2](https://explorers.acloud.pp.ua/cardchain/staking)
=
- **Minimum hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   2| 4GB  | 100GB    |

### Preparing the server

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

# Build 03.10.22
```bash
curl https://get.ignite.com/DecentralCardGame/Cardchain@v0.8! | sudo bash
sudo apt-get install jq
```

`Cardchain version --long | head`
+ version: latest-bf2b2b7b
+ commit: bf2b2b7b07a9fd32ae68f9b72f1d83f608735b5b
    
      Cardchain init <moniker> --chain-id Cardchain

## Create/recover wallet

    Cardchain keys add <walletname>
    Cardchain keys add <walletname> --recover

## Genesis

    wget -O $HOME/.Cardchain/config/genesis.json "https://raw.githubusercontent.com/DecentralCardGame/Testnet1/main/genesis.json"

## Download addrbook

    wget -O $HOME/.Cardchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Crowd%20Control/addrbook.json"


## Minimum gas price/Peers/Seeds

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ubpf\"/;" ~/.Cardchain/config/app.toml

    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.Cardchain/config/config.toml

    peers="61f05a01167b1aec59275f74c3d7c3dc7e9388d4@45.136.28.158:26658"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.Cardchain/config/config.toml

    seeds=""
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.Cardchain/config/config.toml



### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.Cardchain/config/config.toml

# Create a service file

    sudo tee /etc/systemd/system/Cardchain.service > /dev/null <<EOF
    [Unit]
    Description=Cardchain
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which Cardchain) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

## Start

    sudo systemctl daemon-reload
    sudo systemctl enable Cardchain
    sudo systemctl restart Cardchain
    sudo journalctl -u Cardchain -f -o cat

## Create validator


    Cardchain tx staking create-validator \
    --amount 1000000ubpf \
    --from <walletName> \
    --commission-max-change-rate "0.1" \
    --commission-max-rate "0.2" \
    --commission-rate "0.05" \
    --min-self-delegation "1" \
    --details="" \
    --identity="" \
    --pubkey  $(Cardchain tendermint show-validator) \
    --moniker <moniker> \
    --fees 300ubpf \
    --chain-id Cardchain -y


## Delete node

    sudo systemctl stop Cardchain && \
    sudo systemctl disable Cardchain && \
    rm /etc/systemd/system/Cardchain.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .Cardchain && \
    rm -rf Cardchain && \
    rm -rf $(which Cardchain)

