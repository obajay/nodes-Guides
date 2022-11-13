# Konstellation Mainnet guide
![Konstel (1)](https://user-images.githubusercontent.com/44331529/180598012-bab9dd14-99d9-4db4-b6ca-45644f0ee50a.png)
![Konstel (2)](https://user-images.githubusercontent.com/44331529/180598013-4f5b4103-c7cb-4bf4-892f-ff88ff0034af.png)


[Website](https://konstellation.tech/) \
[Explorer](https://www.mintscan.io/konstellation/validators)
=
- **Minimum hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  4 | 8GB  |  160GB   |

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

## Build 13.11.22
```python
cd $HOME
git clone https://github.com/knstl/konstellation/
cd konstellation
git checkout v0.6.0
make install
```
`knstld version`
+ 0.6.0
```    
knstld init STAVRguide --chain-id darchub
```
## Create/recover wallet

    knstld keys add <walletname>
    knstld keys add <walletname> --recover


## Genesis

    wget -O $HOME/.knstld/config/genesis.json https://raw.githubusercontent.com/Konstellation/konstellation/master/config/genesis.json

### Download config.toml with predefined seeds and persistent peers
    wget -O $HOME/.knstld/config/config.toml https://raw.githubusercontent.com/Konstellation/konstellation/master/config/config.toml


### Set up the minimum gas price $HOME/.knstld/config/app.toml as well as seed and peers

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01udarc\"/;" ~/.knstld/config/app.toml
  
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.knstld/config/config.toml



## Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.knstld/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.knstld/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.knstld/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.knstld/config/app.toml

## Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.knstld/config/config.toml


[Snapshot](https://polkachu.com/tendermint_snapshots/konstellation)
=

## Create a service file

    sudo tee /etc/systemd/system/knstld.service > /dev/null <<EOF
    [Unit]
    Description=knstld mainnet
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which knstld) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535
    
    [Install]
    WantedBy=multi-user.target
    EOF



# Start

    sudo systemctl daemon-reload
    sudo systemctl enable knstld
    sudo systemctl restart knstld
    sudo journalctl -u knstld -f -o cat

## Creating validator
    knstld tx staking create-validator \
    --amount 1000000udarc \
    --from <walletName> \
    --commission-max-change-rate "0.1" \
    --commission-max-rate "0.2" \
    --commission-rate "0.07" \
    --min-self-delegation "1" \
    --details="" \
    --identity="" \
    --pubkey  $(knstld tendermint show-validator) \
    --moniker <moniker> \
    --fees 2000000udarc \
    --chain-id darchub -y



### Delete node

    sudo systemctl stop knstld && \
    sudo systemctl disable knstld && \
    rm /etc/systemd/system/knstld.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .knstld && \
    rm -rf konstellation && \
    rm -rf $(which knstld)






