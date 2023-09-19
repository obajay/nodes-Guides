# MEME mainnet guide
![mm (2)](https://user-images.githubusercontent.com/44331529/180606614-565409bd-fa7e-4e50-990d-b2e47614d172.png)
![mm (1)](https://user-images.githubusercontent.com/44331529/180606616-069f9ce4-ffc4-4c0b-ac08-19e168054991.png)

[EXPLORER 1](https://explorer.stavr.tech/Meme/staking) \
[EXPLORER 2](https://ping.pub/meme/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 200GB    |

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

# Build 15.10.22
```python
cd $HOME
wget https://fix.meme.sx/memed-v1.0.0-fix.tar.gz
tar -xvzf memed-v1.0.0-fix.tar.gz
sudo mv ./memed $HOME/go/bin/

```
```python
memed init STAVRguide --chain-id meme-1
```

## Create/recover wallet

    memed keys add <walletname>
    memed keys add <walletname> --recover

## Genesis

    wget -O $HOME/.memed/config/genesis.json "https://raw.githubusercontent.com/memecosmos/mainnet/main/meme-1/genesis.json"

## Download addrbook
```python
wget -O $HOME/.memed/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Meme/addrbook.json"
```

## Minimum gas price/Peers/Seeds
```python
external_address=$(wget -qO- eth0.me)
peers="fce4cbc9f8a9528fcd06948247025c3316991214@116.203.35.46:26656,8db6d048af7c3cbbded64a13e107deac0ecd4e0b@157.230.58.197:26656,0bff1a09a775f3f48125e2608e5425d9916be9ec@157.230.58.200:26656,f51b8d710dd6a556694a5bd414c0e21753027b95@188.166.97.38:26656,7f8d0d370ea72608fa74d0b6698a7979ab510449@188.166.104.46:26656,bbce4f689582db49d7a93cb2baf94d95aa72f43b@137.184.13.23:26656,81ca4565e35d3c3f9cf6cf6d8d1fe7e6c4a2e490@207.148.2.119:26656,1e2a4e7c513d1ba267fe2e689d4dfe6d6105f644@155.138.255.208:26656"
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.memed/config/config.toml
sed -i -E 's/minimum-gas-prices = \"\"/minimum-gas-prices = \"0.025umeme\"/g' ~/.memed/config/app.toml
```
### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.memed/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.memed/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.memed/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.memed/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.memed/config/config.toml

[SNAPSHOT](https://polkachu.com/tendermint_snapshots/meme)


# Create a service file

    sudo tee /etc/systemd/system/memed.service > /dev/null <<EOF
    [Unit]
    Description=memed mainnet
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which memed) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

## Start

    sudo systemctl daemon-reload && \ 
    sudo systemctl enable memed && \
    sudo systemctl restart memed && \
    sudo journalctl -u memed -f -o cat

## Create validator


    memed tx staking create-validator \
    --amount 1000000umeme \
    --from <walletName> \
    --commission-max-change-rate "0.1" \
    --commission-max-rate "0.2" \
    --commission-rate "0.05" \
    --min-self-delegation "1" \
    --details="" \
    --identity="" \
    --pubkey  $(memed tendermint show-validator) \
    --moniker <moniker> \
    --gas 300000 \
    --fees 10000umeme \
    --chain-id meme-1 -y


## Delete node
    sudo systemctl stop memed && \
    sudo systemctl disable memed && \
    rm /etc/systemd/system/memed.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .memed && \
    rm -rf meme && \
    rm -rf $(which memed)

