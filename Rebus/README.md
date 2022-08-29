# Guide Rebus 
![reb (1)](https://user-images.githubusercontent.com/44331529/182308882-ede61e24-d692-43b9-b49f-ddb283d0711e.png)
![reb (2)](https://user-images.githubusercontent.com/44331529/182308884-d53eae18-c1d0-4a5a-8d75-8518eb30eb73.png)


[Website](https://www.rebuschain.com/)
=
[EXPLORER 1](https://exp.nodeist.net/Rebus/staking) \
[EXPLORER 2](https://rebus.explorers.guru/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4| 8GB  | 160GB    |
# 1) Auto_install script
```bash
wget -O rebuss https://raw.githubusercontent.com/obajay/nodes-Guides/main/Rebus/rebuss && chmod +x rebuss && ./rebuss
```
# 2) Manual installation

### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y

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

# Binary   01.08.22
```console 
cd $HOME
git clone https://github.com/rebuschain/rebus.core.git 
cd rebus.core
git checkout testnet
make install
```
`rebusd version --long | head`
+ version: testnet.6f73acac323e89b6b1f7b38aa1ee884b39234e75
+ commit: 6f73acac323e89b6b1f7b38aa1ee884b39234e75

## Initialisation
```console
rebusd init <moniker> --chain-id reb_3333-1
```
## Add wallet
```console
rebusd keys add <walletName>
rebusd keys add <walletName> --recover
```
# Genesis
```console
curl https://raw.githubusercontent.com/rebuschain/rebus.testnet/master/rebus_3333-1/genesis.json > ~/.rebusd/config/genesis.json
```

`sha256sum $HOME/.rebusd/config/genesis.json`
- d382339b5187693ef2e57ff4f33c571ee9bb238ce9fcd68ca99c02116576c41b

### Pruning (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.rebusd/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.rebusd/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.rebusd/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.rebusd/config/app.toml

### Indexer (optional)
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.rebusd/config/config.toml

### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0arebus\"/" ~/.rebusd/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.rebusd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.rebusd/config/config.toml

peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.rebusd/config/config.toml

seeds="a6d710cd9baac9e95a55525d548850c91f140cd9@3.211.101.169:26656,c296ee829f137cfe020ff293b6fc7d7c3f5eeead@54.157.52.47:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.rebusd/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.rebusd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.rebusd/config/config.toml
```

## Download addrbook
```console
wget -O $HOME/.rebusd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Rebus/addrbook.json"
```

# Create a service file
```console
sudo tee /etc/systemd/system/rebusd.service > /dev/null <<EOF
[Unit]
Description=rebus
After=network-online.target

[Service]
User=$USER
ExecStart=$(which rebusd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

# Start node (one command)
```console
sudo systemctl daemon-reload && \
sudo systemctl enable rebusd && \
sudo systemctl restart rebusd && \
sudo journalctl -u rebusd -f -o cat
```

## Create validator
```
rebusd tx staking create-validator \
--amount 1000000arebus \
--from <walletname> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(rebusd tendermint show-validator) \
--moniker <Moniker> \
--chain-id reb_3333-1 \
--gas 300000 \
-y
```

### Delete node (one command)
```
sudo systemctl stop rebusd && \
sudo systemctl disable rebusd && \
rm /etc/systemd/system/rebusd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .rebusd && \
rm -rf rebus.core && \
rm -rf $(which rebusd)
```
