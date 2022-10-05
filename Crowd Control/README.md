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

peers="72b662370d2296a22cad1eecbe447012dd3c2a89@65.21.151.93:36656,b17b995cf2fcff579a4b4491ca8e05589c2d8627@195.54.41.130:36656,d692726a2bdeb0e371b42ef4fa6dfaa47a1c5ad4@38.242.250.15:26656,f1d8bede57e24cb6e5258da1e4f17b1c5b0a0ca3@173.249.45.161:26656,959f9a742058ff591a5359130a392bcccf5f11a5@5.189.165.127:18656,56ff9898493787bf566c68ede80febb76a45eedc@23.88.77.188:20004,96821b39e381e293a251c860c58a2d9e85435363@49.12.245.142:13656,638240b94ac3da7d8c8df8ae4da72a7d920acf2a@173.212.245.44:26656,b41f7ce40c863ee7e20801e6cd3a97237a79114a@65.21.53.39:16656,5d2bb1fed3f93aed0ba5c96bff4b0afb31d9501d@130.185.119.10:26656"
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
sudo rm -rf $HOME/.Cardchain/
sudo rm -rf Testnet
sudo rm /usr/local/bin/Cardchain
```
