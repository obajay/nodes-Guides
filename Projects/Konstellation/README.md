<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 296600.2 | 0.00140542 | 416.84788861 |

<!-- END_TABLE -->























[🔥OUR VALIDATOR🔥](https://restake.app/konstellation/darcvaloper1krlfcngvstzxdy84v0vsfydefmju9wuhdnq03j)
=

# Konstellation Mainnet guide
![Konstel (1)](https://user-images.githubusercontent.com/44331529/180598012-bab9dd14-99d9-4db4-b6ca-45644f0ee50a.png)
![Konstel (2)](https://user-images.githubusercontent.com/44331529/180598013-4f5b4103-c7cb-4bf4-892f-ff88ff0034af.png)


[Website](https://konstellation.tech/) \
[EXPLORER 1](https://explorer.stavr.tech/Konstellation/staking) \
[EXPLORER 2](https://www.mintscan.io/konstellation/validators)
=
- **Minimum hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  4 | 8GB  |  160GB   |

# 1) Auto_install script
```python
wget -O konstm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Konstellation/konstm && chmod +x konstm && ./konstm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 18.1 (one command)

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

## Build 08.12.22
```python
cd $HOME
git clone https://github.com/knstl/konstellation/
cd konstellation
git checkout v0.6.2
make build
cd build
mv knstld $HOME/go/bin/
```
*******🟢UPDATE🟢******* 08.12.22
```python
cd konstellation
git fetch --all
git checkout v0.6.2
make build
cd build
mv knstld $(which knstld)
sudo systemctl restart knstld && sudo journalctl -u knstld -f -o cat
```

`knstld version`
+ 0.6.2
```    
knstld init STAVRguide --chain-id darchub
```
## Create/recover wallet

    knstld keys add <walletname>
    knstld keys add <walletname> --recover


## Genesis

    wget -O $HOME/.knstld/config/genesis.json https://raw.githubusercontent.com/Konstellation/konstellation/master/config/genesis.json

### Set up the minimum gas price $HOME/.knstld/config/app.toml as well as seed and peers

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01udarc\"/;" ~/.knstld/config/app.toml
  
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.knstld/config/config.toml


## Pruning
```python
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.knstld/config/app.toml 
```


## Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.knstld/config/config.toml


[Snapshot](https://polkachu.com/tendermint_snapshots/konstellation)
=

## Download addrbook
```python
wget -O $HOME/.knstld/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Konstellation/addrbook.json"
```


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

    sudo systemctl stop knstld
    sudo systemctl disable knstld
    rm /etc/systemd/system/knstld.service
    sudo systemctl daemon-reload
    cd $HOME
    rm -rf .knstld
    rm -rf konstellation
    rm -rf $(which knstld)






