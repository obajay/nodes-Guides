# Evmos Mainnet guide
![Evmos (1)](https://user-images.githubusercontent.com/44331529/180597649-e0fdcf22-9c86-4c7a-9133-af576fc7631d.png)
![Evmos (2)](https://user-images.githubusercontent.com/44331529/180597656-9b765fb8-ef2c-41cc-977f-cdc7c51a6c3a.png)

[Website](https://evmos.org/) \
[EXPLORER](https://www.mintscan.io/evmos/validators)
=
- **Recommended hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  16| 128GB| 2TB SSD/NVMe |

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
    git clone https://github.com/evmos/evmos
    cd evmos
    git checkout v6.0.1
    make install
    evmosd version
    6.0.1

    evmosd init <moniker-name> --chain-id evmos_9001-2
    
## Create/recover wallet
    evmosd keys add <walletname>
    evmosd keys add <walletname> --recover
##### when creating, do not forget to write down the seed phrase    
## Genesis
    wget https://archive.evmos.org/mainnet/genesis.json
    mv genesis.json ~/.evmosd/config/    
## Set up the minimum gas price $HOME/.evmosd/config/app.toml as well as seed and peers
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0aevmos\"/;" ~/.evmosd/config/app.toml

    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.evmosd/config/config.toml

    peers="d8ac979da3dbe2f796e2344616096160dc5cfdc1@164.92.191.127:26656,d5d418256279900c3d1fbf2137ce7142d6f6c682@65.108.139.20:26656,1915b0217865b968646768e2761a8669d5e24bd5@65.108.44.149:26656,1a7bee67d6337d09380b824b952872bdc5dca86f@38.242.194.56:26656,b02259a11e4ee46b29668cfc957e530022a3fca1@62.171.142.145:26656,cc321917ce82b6c541c687420ad5ae0b4b5e055a@144.76.224.246:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.evmosd/config/config.toml
    seeds=""
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.evmosd/config/config.toml 
    
## Pruning (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.evmosd/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.evmosd/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.evmosd/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.evmosd/config/app.toml
## Indexer (optional)    
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.evmosd/config/config.toml    
    
## Create a service file
    sudo tee /etc/systemd/system/evmosd.service > /dev/null <<EOF
    [Unit]
    Description=evmos
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which evmosd) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF
    
 [Snapshot](https://polkachu.com/tendermint_snapshots/evmos)    (optional) \
    or \
 [RPC](https://nodejumper.io/evmos/sync) (optional)
 
# Start
    sudo systemctl daemon-reload
    sudo systemctl enable evmosd
    sudo systemctl restart evmosd
    sudo journalctl -u evmosd -f -o cat
    
    
## Delete node

    sudo systemctl stop evmosd && \
    sudo systemctl disable evmosd && \
    rm /etc/systemd/system/evmosd.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .evmosd && \
    rm -rf evmos && \
    rm -rf $(which evmosd)
