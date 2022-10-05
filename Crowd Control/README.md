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
git clone https://github.com/DecentralCardGame/Testnet && chmod +x ./Testnet/Cardchain_install.sh && chmod +x ./Testnet/Cardchain_remove.sh && ./Testnet/Cardchain_install.sh
```

`Cardchain version --long | head`
+ version: latest-8103a490
+ commit: 8103a49099f2357d56e31aa090ddc1cd42e07bde
    
## Create/recover wallet
```bash
Cardchain keys add <walletname>
Cardchain keys add <walletname> --recover
```

## Download addrbook
```bash
wget -O $HOME/.Cardchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Crowd%20Control/addrbook.json"
```

## Minimum gas price/Peers/Seeds
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ubpf\"/;" ~/.Cardchain/config/app.toml

external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.Cardchain/config/config.toml

peers="2fb0585484acb391ce36edf90ccbe5024e64f6d0@207.180.194.156:46656,bccdf09677fea2709be54625f3d9c184b327e8de@38.242.255.239:46656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.Cardchain/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.Cardchain/config/config.toml
```


### Pruning (optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain/config/app.toml
```
### Indexer (optional)
```bash
ndexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.Cardchain/config/config.toml
```

## Start
```bash
sudo systemctl daemon-reload
sudo systemctl enable Cardchaind.service
sudo systemctl restart Cardchaind.service
sudo journalctl -u Cardchaind.service -f -o cat
```
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
```bash
sudo systemctl stop Cardchaind
sudo rm /etc/systemd/system/Cardchaind.service
sudo rm -r $HOME/.Cardchain/
sudo rm /usr/local/bin/Cardchain
```
