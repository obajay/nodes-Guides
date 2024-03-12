[🔥OUR VALIDATOR🔥](https://restake.app/genesisl1/genesisvaloper1p4n3fy8wqmn4ja0fp4lenaemyzlxrp6ysrhxfj)
=

# Guide Genesisl1
![ge](https://user-images.githubusercontent.com/44331529/184477593-51a56796-6da8-4b8c-a4ec-9de62084b9e2.png)

[EXPLORER 1](https://explorer.stavr.tech/Genesisl1/staking) \
[EXPLORER 2](https://ping.pub/genesisl1/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB  | 200GB    |

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

# Binary 17.02.24
```python 
git clone https://github.com/alpha-omega-labs/genesis-crypto
cd genesis-crypto
git checkout v1.0.0
go mod tidy
make install
```
`genesisd version --long | grep -e version -e commit`
- version:  1.0.0
- commit: 7cb02ad35442e00b7eb24dc06868577223d48141

## Initialisation
```python
genesisd init STAVR_guide --chain-id genesis_29-2
```
## Add wallet
```python
genesisd keys add <walletName>
    OR
genesisd keys add <walletName> --recover
```
# Genesis
```python
wget -O $HOME/.genesis/config/genesis.json "https://raw.githubusercontent.com/alpha-omega-labs/genesis-parameters/main/genesis_29-2/genesis.json"
```

`sha256sum $HOME/.genesis/config/genesis.json`
- b8f6939f3bfaeec666c7efb94a683e9295e40705ef08af14d55014e4fa11757f  genesis.json

### Pruning (optional)
```python
pruning="custom" &&
pruning_keep_recent="100" &&
pruning_keep_every="0" &&
pruning_interval="10" &&
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.genesis/config/app.toml &&
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.genesis/config/app.toml &&
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.genesis/config/app.toml &&
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.genesis/config/app.toml
```
### Indexer (optional)
```python
indexer="null" &&
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.genesis/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"50000000000el1\"/;" ~/.genesis/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.genesis/config/config.toml
peers="3985c968899e7344991ba3589c95b0e6a0ce982c@188.165.211.196:26656,2646a043e1f0c766c5b704463a7d811e100ec7f3@158.69.253.120:26656,0d07fb60f8491f4b53a6b58ae0ce60d4c69be506@135.181.183.88:26656,7757fdee74e8d33ecaa63ead16b3564cb9dea258@85.10.200.11:26656,ef7d81eb8db7ad59b4ce30e022c758cee8dc174f@188.165.202.131:26656,673ec772091d7c4e4dc8af7ed00edea4c8d334ac@65.21.196.125:26656,0d8f14bfcd680a471c4c181590b7a6910544115d@188.40.91.228:26656,0936e624c45ff1ac4089856da2beea148ee6c8de@62.171.183.162:26656,af405a6c392b747aa74704ad0ee8585b8ce164b3@37.187.95.163:26656,0f9ad819318bfa9735603736aa4c6265f666a7d9@5.135.143.103:26656,060585a1cc1fa88b4188a2d94de07b518dc188cf@144.91.84.196:26656,62cb81bad72ed77c776c7fec0547b09bdc5ceb22@158.69.253.103:26656,1d07c049908e614f5d00bf64539581178a2a7f0d@192.99.5.180:26656,be81a20b7134552e270774ec861c4998fabc2969@5.189.128.191:26656,70c201d6568e0ddf1ebe105df06b957cbc255a8b@46.4.108.77:26656,1c41828553d7ed77fb778be9c9c48a8070958744@174.138.180.190:61356,ac8056270101705557e14291dc0c98ef4f65c514@65.109.18.209:26656,75525c6609cf1600d62531b0f4bb2dc4a1f81020@187.85.19.63:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.genesis/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.genesis/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.genesis/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.genesis/config/config.toml
genesisd config chain-id genesis_29-2
sed -i -e "s/^timeout_commit *=.*/timeout_commit = \"10s\"/" $HOME/.genesis/config/config.toml


```

## Download addrbook
```python
wget -O $HOME/.genesis/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Genesisl1/addrbook.json"
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
sudo systemctl daemon-reload && sudo systemctl enable genesisd &&
sudo systemctl restart genesisd && sudo journalctl -u genesisd -f -o cat
```

## Create validator
```python
genesisd tx staking create-validator \
--amount=1000000000000000000el1 \
--pubkey=$(genesisd tendermint show-validator) \
--moniker=STAVR_guide \
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
sudo systemctl stop genesisd
sudo systemctl disable genesisd
rm /etc/systemd/system/genesisd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .genesis
rm -rf genesisd
rm -rf $(which genesisd)
```
