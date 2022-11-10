# Guide Galaxy mainnet
![GAL](https://user-images.githubusercontent.com/44331529/182396081-d701cfac-63c1-489f-80e1-c2df72c73863.png)

[Website](https://galaxychain.zone/)
=
[EXPLORER 1](https://explorer.postcapitalist.io/galaxy/staking) \
[EXPLORER 2](https://galaxychain.zone/stake)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 250GB    |
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

# Binary   11.11.22
```bash 
cd $HOME
git clone https://github.com/galaxies-labs/galaxy
cd galaxy
git checkout v1.2.0
make install
```

## Initialisation
```console
galaxyd init STAVRguide --chain-id galaxy-1
galaxyd config chain-id galaxy-1

```
## Add wallet
```console
galaxyd keys add <walletName>
galaxyd keys add <walletName> --recover
```
# Genesis
```console
wget -O $HOME/.galaxy/config/genesis.json "https://media.githubusercontent.com/media/galaxies-labs/networks/main/galaxy-1/genesis.json"
```

`sha256sum $HOME/.galaxy/config/genesis.json`
- 2003cfaca53c3f9120a36957103fbbe6562d4f6c6c50a3e9502c49dbb8e2ba5b

### Pruning (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.galaxy/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.galaxy/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.galaxy/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.galaxy/config/app.toml

### Indexer (optional)
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.galaxy/config/config.toml

### Set up the Peers/Seeds/Filter peers/MaxPeers
```console
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.galaxy/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.galaxy/config/config.toml

peers="bf446887a7a00c8babfeba2f92ba569a206a3ea7@65.108.71.140:26676,1e9ee1911298a15128c8485ea47b18be08939e01@136.244.29.116:38656,a4bd8fed416aa29d9cc061e2b9dffa7f4b679c91@65.21.131.144:30656,801f4e17769bd2ee02b27720d901a42cb8d052ea@65.108.192.3:24656,8fc2d8c2fadd278eae617a9c2a2f008e01e8ef68@206.246.71.251:26656,10f7caa39969dc36450b138848a06e7deabe6fed@95.111.244.128:26656,cd8fd9e1677c701015b8909116f88974028cd0b4@203.135.141.28:26656,b4b6f1563f2891ed5735d6133d78fc7c17ce12d0@185.234.69.139:26656,5b3fd251b74e6af11f4c71d420fd1837f4869e85@45.33.62.64:26656,51b3263a333de94198fe4c4d819b48fbd107f93a@5.9.13.234:26356,e21bf32eaedee13d8dc240baacf23fee97a8edac@141.94.141.144:43656,8b447bd4fa1e56d8252538a6e23573e5e78924fa@161.97.155.94:26656,8d059154ea0a6e25c5695a1e163e601482769604@95.217.207.236:31256,7ded7314f57a078076507d7b291e100ad2dc158b@65.108.41.172:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.galaxy/config/config.toml

seeds="574e8402e255f895680db2904168724258fd6ff8@13.125.60.249:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.galaxy/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.galaxy/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.galaxy/config/config.toml
```

## Download addrbook
```console
wget -O $HOME/.galaxy/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Galaxy/addrbook.json"
```

# Create a service file
```console
sudo tee /etc/systemd/system/galaxyd.service > /dev/null <<EOF
[Unit]
Description=galaxy
After=network-online.target

[Service]
User=$USER
ExecStart=$(which galaxyd) start
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
sudo systemctl enable galaxyd && \
sudo systemctl restart galaxyd && \
sudo journalctl -u galaxyd -f -o cat
```

## Create validator
```
galaxyd tx staking create-validator \
--amount 1000000uglx \
--from <walletname> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(galaxyd tendermint show-validator) \
--moniker <Moniker> \
--chain-id galaxy-1 \
-y
```

### Delete node (one command)
```
sudo systemctl stop galaxyd && \
sudo systemctl disable galaxyd && \
rm /etc/systemd/system/galaxyd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .galaxy && \
rm -rf galaxy && \
rm -rf $(which galaxyd)
```
