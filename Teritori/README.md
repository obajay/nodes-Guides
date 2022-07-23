# Teritori testnet guide
![tert](https://user-images.githubusercontent.com/44331529/180614436-1041172a-0b1e-4df3-85b7-3d18899f3e43.png)

[EXPLORER](https://explorer.ericet.xyz/teritori/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   2| 4GB  | 100GB    |

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

    git clone https://github.com/TERITORI/teritori-chain
    cd teritori-chain
    git checkout teritori-testnet-v2
    make install

    teritorid init <moniker> --chain-id teritori-testnet-v2
    teritorid config chain-id teritori-testnet-v2
    
## Create/recover wallet

    teritorid keys add <walletname>
    teritorid keys add <walletname> --recover

### when creating, do not forget to write down the seed phrase

# Genesis

    wget https://raw.githubusercontent.com/TERITORI/teritori-chain/teritori-testnet-v2/genesis/genesis.json -O $HOME/.teritorid/config/genesis.json

## Seeds,peers and gas price
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0utori\"/;" ~/.teritorid/config/app.toml
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.teritorid/config/config.toml

    peers="0dde2ae55624d822eeea57d1b5e1223b6019a531@176.9.149.15:26656,4d2ea61e6195ee4e449c1e6132cabce98f7d94e1@5.9.40.222:26656,bceb776975aab62bcfd501969c0e1a2734ed7c2e@176.9.19.162:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.teritorid/config/config.toml
    seeds=""
    sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.teritorid/config/config.toml

## Download addrbook

    wget -O $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Teritori/addrbook.json"


## Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.teritorid/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.teritorid/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.teritorid/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.teritorid/config/app.toml

## Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.teritorid/config/config.toml

# Create a service file

    sudo tee /etc/systemd/system/teritorid.service > /dev/null <<EOF
    [Unit]
    Description=Teritorid
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which teritorid) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF


    
# Start

    sudo systemctl daemon-reload
    sudo systemctl enable teritorid
    sudo systemctl restart teritorid
    sudo journalctl -u teritorid -f -o cat

[DISCORD FAUCET](https://discord.gg/zzJEmR8nhr)

## Create validator
    teritorid tx staking create-validator \
    --amount=1000000utori \
    --pubkey=$(teritorid tendermint show-validator) \
    --moniker="<moniker>" \
    --identity="" \
    --details="" \
    --website="" \
    --chain-id="teritori-testnet-v2" \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.1" \
    --min-self-delegation="1" \
    --fees 500utori \
    --from=<walletname> -y


# Delete node
    sudo systemctl stop teritorid && \
    sudo systemctl disable teritorid && \
    rm /etc/systemd/system/teritorid.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .teritorid && \
    rm -rf teritori-chain && \
    rm -rf $(which teritorid)



