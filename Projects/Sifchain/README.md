<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 19053784.5 | 0.00022126 | 4215.840372435145668190 |

<!-- END_TABLE -->



































































































[🔥OUR VALIDATOR🔥](https://restake.app/sifchain/sifvaloper1k5ypsesvvfga6pxjdxggaph97ywwf4l4mw0mqp)
=

# Sifchain mainnet guide
![sifchain](https://user-images.githubusercontent.com/44331529/190616339-2de8f67c-4818-4a99-8b5b-6b6e713fd023.png)

[Website](https://sifchain.network/)
=
[EXPLORER 1](https://explorer.stavr.tech/Sifchain/staking) \
[EXPLORER 2](https://www.mintscan.io/sifchain/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 260GB    |


# 1) Auto_install script
```pytohn
wget -O sifchain https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sifchain/sifchain && chmod +x sifchain && ./sifchain
```

# 2) Manual installation

### Preparing the server
```pytohn
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 18.5
```pytohn
cd $HOME
ver="1.18.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

# Build 18.12.23
```python
cd $HOME
git clone https://github.com/Sifchain/sifnode
cd $HOME/sifnode
git checkout v1.4.0
make install
```
`sifnoded version --long`
- version: v1.4.0
- commit: 191a7e41d0bd0ab50e1535b3951ce16697dd0090

```python
sifnoded init STAVR_guide --chain-id sifchain-1
```    

## Create/recover wallet
```python
sifnoded keys add <walletname>
sifnoded keys add <walletname> --recover
```

## Download Genesis

```python
cd "${HOME}"/.sifnoded/config
wget -O genesis.json.gz https://raw.githubusercontent.com/Sifchain/networks/master/betanet/sifchain-1/genesis.json.gz
gunzip genesis.json.gz
```

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i "s/persistent_peers =.*/persistent_peers = \"0d4981bdaf4d5d73bad00af3b1fa9d699e4d3bc0@44.235.108.41:26656,bcc2d07a14a8a0b3aa202e9ac106dec0bef91fda@13.55.247.60:26656,663dec65b754aceef5fcccb864048305208e7eb2@34.248.110.88:26656,0120f0a48e7e81cc98829ef4f5b39480f11ecd5a@52.76.185.17:26656,6535497f0152293d773108774a705b86c2249a9c@44.238.121.65:26656,fdf5cffc2b20a20fab954d3b6785e9c382762d14@34.255.133.248:26656,8c240f71f9e060277ce18dc09d82d3bbb05d1972@13.211.43.177:26656,9fbcb6bd5a7f20a716564157c4f6296d2faf5f64@18.138.208.95:26656\"/g" "${HOME}"/.sifnoded/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0stake\"/;" ~/.sifnoded/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sifnoded/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sifnoded/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sifnoded/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.sifnoded/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.sifnoded/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.sifnoded/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.sifnoded/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.sifnoded/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.sifnoded/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sifnoded/config/config.toml
``` 
## Download addrbook
```python
wget -O $HOME/.sifnoded/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sifchain/addrbook.json"
```

# StateSync
```python
SOOON
```
# SnapShot (~2.6 GB) updated every 15 hours
```python
SOOON
```

# Create a service file
```python
sudo tee /etc/systemd/system/sifnoded.service > /dev/null <<EOF
[Unit]
Description=sifchain
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sifnoded) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload &&
sudo systemctl enable sifnoded &&
sudo systemctl restart sifnoded && sudo journalctl -u sifnoded -f -o cat
```

### Create validator
```python
sifnoded tx staking create-validator \
--amount=1000000rowan \
--broadcast-mode=block \
--pubkey=`sifnoded tendermint show-validator` \
--moniker=STAVRguide \
--commission-rate="0.1" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--from=<walletName> \
--chain-id=sifchain-1 \
--gas=auto -y
```

## Delete node
```python
sudo systemctl stop sifnoded
sudo systemctl disable sifnoded
rm /etc/systemd/system/sifnoded.service
sudo systemctl daemon-reload
cd $HOME
rm -rf sifnode
rm -rf .sifnoded
rm -rf $(which sifnoded)
```

