# Guide Another-1 testnet

[EXPLORER 1](https://www.skynetexplorers.com/DWS/staking) \
[EXPLORER 2](https://test-anone.zenscan.io/validators.php)
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
git clone https://github.com/notional-labs/anone.git
cd anone
git checkout testnet-1.0.3
make install
```
`anoned version version --long | head`
+ version: testnet-1.0.3
+ commit: 2da1c958096654a7f9bbd0b5bf522c5260d04637

## Initialisation
```console
anoned init <moniker> --chain-id anone-testnet-1
anoned config chain-id anone-testnet-1
```
## Add wallet
```console
anoned keys add <walletName>
anoned keys add <walletName> --recover
```
# Genesis
```console
wget -O $HOME/.anone/config/genesis.json "https://raw.githubusercontent.com/notional-labs/anone/master/networks/testnet-1/genesis.json"
```

`sha256sum $HOME/.anone/config/genesis.json`
- ba7bea692350ca8918542a26cabd5616dbebe1ff109092cb1e98c864da58dabf

### Pruning (optional)
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.anone/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.anone/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.anone/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.anone/config/app.toml
```

### Indexer (optional)
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.anone/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uan1\"/;" ~/.anone/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.anone/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.anone/config/config.toml

peers="2b540c43d640befc35959eb062c8505612b7d67f@another1-testnet.nodejumper.io:26656,a0ff256334e6781972beec33739123fd852153a3@95.217.121.243:2280,35c14ef98034511e716504c6b7aa9d9ed416a75f@62.141.41.220:26656,4d2099cb772f639e7e2936f9f9f2a9a85ab35e62@173.249.7.49:26656,82ba6b00244af1b1fee3dc415d398188de40217b@75.119.135.167:26656,75e21f3f515294caadaed054297b591e7aff1ff0@173.212.223.37:26656,6abd85339523371ceb44ecc45c17b24836e4a13d@209.126.7.201:26656,c52aa7de58b29d93b17d09a373e6adb2eb29f5f1@144.126.138.48:26656,7130dc7f837215eba6429c752b606f2165f72463@207.244.246.217:26656,a6090021754819f1e055be8ff814c1fdb3ab5e51@144.126.140.91:26656,1fcf5a1cbdec73092ef3bfe3944fbfc6d240c6d6@185.230.138.141:26656,c760ef73579bc95fd15367f81a015113bd79e675@65.21.129.95:26656,b3e85b210dce19c7d5682a836ec7287a96a9d4c0@159.223.34.123:26656,05a4c982b3bc5a4dff9508c0b0d9d401357018f6@144.126.137.231:26656,74c334436da9e0e6d17ac083653601649aad4498@185.209.229.135:26656,5c2b1c4deb14501871c773e8c6c41bbcfe853471@207.244.243.245:26656,05c242cf520fc35b1ddc3536d55e6ce25cdc4117@161.97.126.98:26656,5bd7cef7ff9b50847532f311e17aa74e7e45a56d@135.181.214.164:26656,0d83b9159805de0aa0a3a0c213362de565dad64e@95.217.198.243:26656,300507b829a8befac579dd0c7357851a188ab973@144.126.138.115:26656,229b31707536e32447a94edf63f9e0c999e31097@95.111.239.233:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.anone/config/config.toml

seeds="49a49db05e945fc38b7a1bc00352cafdaef2176c@95.217.121.243:2280,80f0ef5d7c432d2bae99dc8437a9c3db464890cd@65.108.128.139:2280,3afac655e3be5c5fc4a64ec5197346ffb5a855c1@49.12.213.105:2280"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.anone/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.anone/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.anone/config/config.toml
```

## Download addrbook
```console
wget -O $HOME/.anone/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Another-1/addrbook.json"
```

# Create a service file
```console
sudo tee /etc/systemd/system/anoned.service > /dev/null <<EOF
[Unit]
Description=anone
After=network-online.target

[Service]
User=$USER
ExecStart=$(which anoned) start
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
sudo systemctl enable anoned && \
sudo systemctl restart anoned && \
sudo journalctl -u anoned -f -o cat
```

## Create validator
```
anoned tx staking create-validator \
--amount 1000000uan1 \
--from <walletname> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(anoned tendermint show-validator) \
--moniker <Moniker> \
--chain-id anone-testnet-1 \
--gas="auto" \
--fees 100udws
-y
```

### Delete node (one command)
```
sudo systemctl stop anoned && \
sudo systemctl disable anoned && \
rm /etc/systemd/system/anoned.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .anone && \
rm -rf anone && \
rm -rf $(which anoned)
```
