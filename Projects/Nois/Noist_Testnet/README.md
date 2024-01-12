# Nois testnet guide V005

![nnnoi](https://user-images.githubusercontent.com/44331529/191945004-1227fef0-a215-44f1-bcab-854acd66de00.png)

[Website](https://nois.network/)
=
[EXPLORER 1](http://explorer.stavr.tech/Nois-Testnet/staking) \
[EXPLORER 2](https://testnet.ping.pub/nois/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 160GB    |


# 1) Auto_install script
```Python
wget -O nois https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nois/Noist_Testnet/nois && chmod +x nois && ./nois
```

# 2) Manual installation

### Preparing the server

```Python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 19 

```Python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 12.01.24 
```Python
cd $HOME
git clone https://github.com/noislabs/noisd
cd noisd
git checkout v1.0.5
make install
```
`noisd version --long | grep -e commit -e version`
- version: 1.0.5
- commit: 1e7b65f785b43e9b389ff7be058d935677fdaf78

```Python
noisd init STAVR_guide --chain-id nois-testnet-005
noisd config chain-id nois-testnet-005
```    

## Create/recover wallet
```Python
noisd keys add <walletname>
noisd keys add <walletname> --recover
```

## Download Genesis

```Python
wget -O $HOME/.noisd/config/genesis.json "https://raw.githubusercontent.com/noislabs/networks/main/nois-testnet-005/genesis.json"
```
`sha256sum $HOME/.noisd/config/genesis.json`
+ 1647cabd044110d6beda2fc4a32fb50dfbb5babff74bdf41503c58123ded3389

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```Python
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.noisd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.noisd/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml
seeds="bf07906c7cf0f23606c83be15624be2c67b3929c@139.59.154.47:17356"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.noisd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.noisd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.noisd/config/config.toml
export CONFIG_DIR="$HOME/.noisd/config"
# Update app.toml
sed -i 's/minimum-gas-prices =.*$/minimum-gas-prices = "0.05unois"/' $CONFIG_DIR/app.toml

```
### Pruning (optional)
```Python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.noisd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.noisd/config/app.toml
```
### Indexer (optional) 
```Python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.noisd/config/config.toml
```

## Download addrbook
```Python
wget -O $HOME/.noisd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nois/Noist_Testnet/addrbook.json"
```

# StateSync
```Python
SOOON
```

# Snaphot 
```Python
SPPPN
```


# Create a service file
```Python
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=noisd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noisd) start --home $HOME/.noisd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```Python
sudo systemctl daemon-reload
sudo systemctl enable noisd
sudo systemctl restart noisd && sudo journalctl -u noisd -f -o cat
```

### Create validator
```Python
noisd tx staking create-validator \
--amount=99000000unois \
--pubkey=$(noisd tendermint show-validator) \
--moniker=STAVR_guide \
--chain-id=nois-testnet-005 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--from=<wallet> \
--fees=16000unois \
--gas=300000 \
-y
```
[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Nois)
=

## Delete node
```Python
sudo systemctl stop noisd
sudo systemctl disable noisd
rm /etc/systemd/system/noisd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf full-node
rm -rf .noisd
rm -rf $(which noisd)
```

