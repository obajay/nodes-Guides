# Guide Genesisl1

![ge](https://user-images.githubusercontent.com/44331529/184477593-51a56796-6da8-4b8c-a4ec-9de62084b9e2.png)

[EXPLORER 1](https://explorer.stavr.tech/genesisl1/staking) \
[EXPLORER 2](https://ping.pub/genesisl1/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   8| 16GB  | 200GB    |

# 1) Auto_install script

```python
wget -O genesisx https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Genesisl1/genesisx && chmod +x genesisx && ./genesisx
```
# 2) Manual installation
### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
```
## GO 1.20.5 (one command)
```python
ver="1.20.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

# Binary   01.07.23
```python 
git clone https://github.com/alpha-omega-labs/genesisd
cd genesisd
make install
```
`genesisd version --long | head`
- version:  0.3.1
- commit: 5278bc80caae12ff2aba4b0d9742adfc9f30df9a 

## Initialisation
```python
genesisd init STAVRguide --chain-id genesis_29-2
```
## Add wallet
```python
genesisd keys add <walletName>
genesisd keys add <walletName> --recover
```
# Genesis
```python
wget -O $HOME/.genesisd/config/genesis.json "https://github.com/alpha-omega-labs/genesisd/raw/neolithic/genesis_29-1-state/genesis.json"
```

`sha256sum $HOME/.genesisd/config/genesis.json`
- b8f6939f3bfaeec666c7efb94a683e9295e40705ef08af14d55014e4fa11757f  genesis.json

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.genesisd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.genesisd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.genesisd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.genesisd/config/app.toml
```
### Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.genesisd/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025el1\"/;" ~/.genesisd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.genesisd/config/config.toml
peers="551cb3d41d457f830d75c7a5b8d1e00e6e5cbb91@135.181.97.75:26656,5082248889f93095a2fd4edd00f56df1074547ba@146.59.81.204:26651,36111b4156ace8f1cfa5584c3ccf479de4d94936@65.21.34.226:26656,c23b3d58ccae0cf34fc12075c933659ff8cca200@95.217.207.154:26656,37d8aa8a31d66d663586ba7b803afd68c01126c4@65.21.134.70:26656,d7d4ea7a661c40305cab84ac227cdb3814df4e43@139.162.195.228:26656,be81a20b7134552e270774ec861c4998fabc2969@genesisl1.3ventures.io:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.genesisd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.genesisd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.genesisd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.genesisd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.genesisd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Genesisl1/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/genesisd.service > /dev/null <<EOF
[Unit]
Description=genesisd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which genesisd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## SnapShot  
```python
SOOON
```

# Start node (one command)
```python
sudo systemctl daemon-reload && \
sudo systemctl enable genesisd && \
sudo systemctl restart genesisd && \
sudo journalctl -u genesisd -f -o cat
```

## Create validator
```python
genesisd tx staking create-validator \
--amount=1000000el1 \
--pubkey=$(genesisd tendermint show-validator) \
--moniker=<moniker> \
--chain-id=genesis_29-2 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees=500el1 \
--from=<walletName> \
--identity="" \
--website="" \
--details="" \
-y
```

### Delete node (one command)
```python
sudo systemctl stop genesisd && \
sudo systemctl disable genesisd && \
rm /etc/systemd/system/genesisd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .genesisd && \
rm -rf genesisd && \
rm -rf $(which genesisd)
```
