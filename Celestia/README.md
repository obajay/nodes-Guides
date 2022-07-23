# Celestia testnet guide
![Celestia](https://user-images.githubusercontent.com/44331529/180597147-c00ebd04-de42-476e-bc8c-1142479a839b.png)


[EXPLORER](https://celestia.explorers.guru/validators)
===
- **Minimum hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mamaki    |   4| 8GB  | 160GB    |

### Preparing the server

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## GO 18.3 (one command)

    ver="1.18.3" && \
    cd $HOME && \
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
    sudo rm -rf /usr/local/go && \
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
    rm "go$ver.linux-amd64.tar.gz" && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
    source $HOME/.bash_profile && \
    go version

# Build 13.07.22  (one command)
    git clone https://github.com/celestiaorg/celestia-app --branch v0.6.0 && \
    cd celestia-app && \
    git fetch && \
    git checkout v0.6.0 && \
    make install
`celestia-appd version`
+ 0.6.0

      celestia-appd init <Moniker> --chain-id mamaki
      celestia-appd config chain-id mamaki

## Create/recover wallet

    celestia-appd keys add <walletname>
    celestia-appd keys add <walletname> --recover

## Download Genesis
    wget -O $HOME/.celestia-app/config/genesis.json "https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/genesis.json"
`sha256sum ~/.celestia-app/config/genesis.json`
+ 48747645055290a91a2671d51da399e0921fea93aa1eb0d2a54bab5c43e8a5aa

## Download addrbook

    wget -O $HOME/.celestia-app/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Celestia/addrbook.json"

## Peers/Seeds/Min-Gas and etc

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0utia\"/;" ~/.celestia-app/config/app.toml
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external-address *=.*/external-address = \"$external_address:26656\"/" $HOME/.celestia-app/config/config.toml

    peers="e4429e99609c8c009969b0eb73c973bff33712f9@141.94.73.39:43656,09263a4168de6a2aaf7fef86669ddfe4e2d004f6@142.132.209.229:26656,13d8abce0ff9565ed223c5e4b9906160816ee8fa@94.62.146.145:36656,72b34325513863152269e781d9866d1ec4d6a93a@65.108.194.40:26676,322542cec82814d8903de2259b1d4d97026bcb75@51.178.133.224:26666,5273f0deefa5f9c2d0a3bbf70840bb44c65d835c@80.190.129.50:49656,7145da826bbf64f06aa4ad296b850fd697a211cc@176.57.189.212:26656,5a4c337189eed845f3ece17f88da0d94c7eb2f9c@209.126.84.147:26656,ec072065bd4c6126a5833c97c8eb2d4382db85be@88.99.249.251:26656,cd1524191300d6354d6a322ab0bca1d7c8ddfd01@95.216.223.149:26656,2fd76fae32f587eceb266dce19053b20fce4e846@207.154.220.138:26656,1d6a3c3d9ffc828b926f95592e15b1b59b5d8175@135.181.56.56:26656,fe2025284ad9517ee6e8b027024cf4ae17e320c9@198.244.164.11:26656,fcff172744c51684aaefc6fd3433eae275a2f31b@159.203.18.242:26656,f7b68a491bae4b10dbab09bb3a875781a01274a5@65.108.199.79:20356,6c076056fc80a813b26e24ba8d28fa374cd72777@149.102.153.197:26656,180378bab87c9cecea544eb406fcd8fcd2cbc21b@168.119.122.78:26656,6c076056fc80a813b26e24ba8d28fa374cd72777@149.102.153.197:26656,88fa96d09a595a1208968727819367bd2fe8eabe@164.70.120.56:26656,84133cfde6e5fcaf5915436d56b3eef1d1996d17@45.132.245.56:26656,42b331adaa9ece4c455b92f0d26e3382e46d43f0@161.97.180.20:36656,c8c0456a5174ab082591a9466a6e0cb15c915a65@194.233.85.193:26656,6a62bf1f489a5231ddc320a2607ab2595558db75@154.12.240.49:26656,d0b19e4d133441fd41b4d74ac8de2138313ad49e@195.201.41.137:26656,bf199295d4c142ebf114232613d4796e6d81a8d0@159.69.110.238:26656,a46bbdb81e66c950e3cdbe5ee748a2d6bdb185dd@161.97.168.77:26656"
    sed -i.bak -e "s/^persistent-peers *=.*/persistent-peers = \"$peers\"/" $HOME/.celestia-app/config/config.toml
    sed -i.bak -e "s/^timeout-commit *=.*/timeout-commit = \"25s\"/" $HOME/.celestia-app/config/config.toml
    sed -i.bak -e "s/^skip-timeout-commit *=.*/skip-timeout-commit = false/" $HOME/.celestia-app/config/config.toml
    sed -i.bak -e "s/^mode *=.*/mode = \"validator\"/" $HOME/.celestia-app/config/config.toml
    sed -i.bak -e "s/^use-legacy *=.*/use-legacy = \"true\"/" $HOME/.celestia-app/config/config.toml


### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.celestia-app/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.celestia-app/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.celestia-app/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.celestia-app/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.celestia-app/config/config.toml
    
# SnapShot
    
    cd $HOME
    rm -rf ~/.celestia-app/data
    mkdir -p ~/.celestia-app/data
    SNAP_NAME=$(curl -s https://snaps.qubelabs.io/celestia/ | \
    egrep -o ">mamaki.*tar" | tr -d ">")
    wget -O - https://snaps.qubelabs.io/celestia/${SNAP_NAME} | tar xf - \
    -C ~/.celestia-app/data/
    
# Create a service file

    sudo tee /etc/systemd/system/celestia-appd.service > /dev/null <<EOF
    [Unit]
    Description=celestia-appd
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which celestia-appd) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF


## Start
    sudo systemctl daemon-reload && \
    sudo systemctl enable celestia-appd && \
    sudo systemctl restart celestia-appd && \
    sudo journalctl -u celestia-appd -f -o cat


## Create validator

    celestia-appd tx staking create-validator \
    --chain-id mamaki \
    --commission-rate 0.1 \
    --commission-max-rate 0.2 \
    --commission-max-change-rate 0.1 \
    --min-self-delegation "1000000" \
    --amount 1000000utia \
    --pubkey $(celestia-appd tendermint show-validator) \
    --moniker "<Moniker>" \
    --from <WalletName> \
    --fees 200utia
    
## Delete node (one command)
    sudo systemctl stop celestia-appd && \
    sudo systemctl disable celestia-appd && \
    rm /etc/systemd/system/celestia-appd.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .celestia-app 
    rm -rf celestia-app && \
    rm -rf $(which celestia-appd)
    
