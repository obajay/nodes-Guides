# Guide DWS testnet

![dws (1)](https://user-images.githubusercontent.com/44331529/182409218-5d46916b-fcf2-42fc-8106-bfb1709d36e4.png)
![dws (3)](https://user-images.githubusercontent.com/44331529/182409227-c0c5c15c-5d87-4845-b404-1621aa784d2f.png)
![dws (2)](https://user-images.githubusercontent.com/44331529/182409225-52f077d1-4d23-4835-8353-d5f65320a79b.png)



[Website](https://deweb.services/)
=
[EXPLORER 1](https://www.skynetexplorers.com/DWS/staking) \
[EXPLORER 2](https://dws.explorers.guru/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   2| 4GB  | 100GB    |
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
git clone https://github.com/deweb-services/deweb.git
cd deweb
git checkout v0.2
make install
```
`dewebd version version --long | head`
+ version: 0.2
+ commit: 551192227bb914bc0e2d751ee18043361e7f83f2

## Initialisation
```console
dewebd init <name_node> --chain-id deweb-testnet-1
dewebd config chain-id deweb-testnet-1
```
## Add wallet
```console
dewebd keys add <walletName>
dewebd keys add <walletName> --recover
```
# Genesis
```console
wget -O $HOME/.deweb/config/genesis.json "https://raw.githubusercontent.com/deweb-services/deweb/main/genesis.json"
```

`sha256sum $HOME/.deweb/config/genesis.json`
- 13bf101d673990cb39e6af96e3c7e183da79bd89f6d249e9dc797ae81b3573c2

### Pruning (optional)
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.deweb/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.deweb/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.deweb/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.deweb/config/app.toml
```

### Indexer (optional)
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.deweb/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0udws\"/;" ~/.deweb/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.deweb/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.deweb/config/config.toml

peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.deweb/config/config.toml

seeds="74d8f92c37ffe4c6393b3718ca531da8f0bf0594@seed1.deweb.services:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.deweb/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.deweb/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.deweb/config/config.toml
```

## Download addrbook
```console
wget -O $HOME/.deweb/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/DWS/addrbook.json"
```

# Create a service file
```console
sudo tee /etc/systemd/system/dewebd.service > /dev/null <<EOF
[Unit]
Description=deweb
After=network-online.target

[Service]
User=$USER
ExecStart=$(which dewebd) start
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
sudo systemctl enable dewebd && \
sudo systemctl restart dewebd && \
sudo journalctl -u dewebd -f -o cat
```

## Create validator
```
dewebd tx staking create-validator \
--amount 1000000udws \
--from <walletname> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(dewebd tendermint show-validator) \
--moniker <Moniker> \
--chain-id deweb-testnet-1 \
--gas="auto" \
--fees 100udws
-y
```

### Delete node (one command)
```
sudo systemctl stop dewebd && \
sudo systemctl disable dewebd && \
rm /etc/systemd/system/dewebd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .deweb && \
rm -rf deweb && \
rm -rf $(which dewebd)
```
