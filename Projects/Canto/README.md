<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 71.073 | 0.176778 | 12.564264546258980442 |

<!-- END_TABLE -->



































































































[🔥OUR VALIDATOR🔥](https://restake.app/canto/cantovaloper1tav4ldqxyjhcymdhswxrjrmy69un2yh4vpfhtt)
=


# Guide Canto Mainnet
![canto](https://user-images.githubusercontent.com/44331529/185346490-c8f643a2-8465-432a-90dd-950b0e26957c.png)

[Website](https://canto.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/Canto-Mainnet/staking) \
[EXPLORER 2](https://mainnet.manticore.team/canto/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB  | 260GB    |


# 1)    Auto_install script
```python
wget -O canto https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Canto/canto && chmod +x canto && ./canto
```
# 2)    Manual installation

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
go version
```

# Binary   20.09.23
```python 
git clone https://github.com/Canto-Network/Canto
cd Canto
git checkout v7.0.0
make install
```
`cantod version --long | head`
- version: 7.0.0
- commit: da939e2935d0661eb68449bfd64122efce6f6871

## Initialisation
```python
cantod init STAVR_Guide --chain-id canto_7700-1
```
## Add wallet
```python
cantod keys add <walletName>
cantod keys add <walletName> --recover
```
# Genesis
```python
wget -O genesis.json https://snapshots.polkachu.com/genesis/canto/genesis.json --inet4-only
mv genesis.json ~/.cantod/config
```

`sha256sum $HOME/.cantod/config/genesis.json`
- 5048ba449ae348682fd86840452e88bd0812316279697c04ad288a9059f12f59  genesis.json

### Pruning (optional) one command
```python
pruning="custom" && \
pruning_keep_recent="1000" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cantod/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cantod/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.cantod/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cantod/config/app.toml
```
### Indexer (optional) one command
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.cantod/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025acanto\"/;" ~/.cantod/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.cantod/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.cantod/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:15556"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.cantod/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.cantod/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.cantod/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.cantod/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Canto/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/cantod.service > /dev/null <<EOF
[Unit]
Description=cantod
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cantod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

# SnapShot
```python
SOON
```
# StateSync
```python
SOON
```

# Start node (one command)
```python
sudo systemctl daemon-reload
sudo systemctl enable cantod
sudo systemctl restart cantod && sudo journalctl -u cantod -f -o cat
```

## Create validator
```python
cantod tx staking create-validator \
--amount=1000000000000000000acanto \
--pubkey=$(cantod tendermint show-validator) \
--moniker=STAVR_Guide \
--chain-id=canto_7700-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees 500acanto \
--from=<walletName> \
--identity="" \
--website="" \
--details="" \
-y
```

### Delete node (one command)
```python
sudo systemctl stop cantod
sudo systemctl disable cantod
rm /etc/systemd/system/cantod.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .cantod
rm -rf Canto
rm -rf $(which cantod)
```

#
### Sync Info
```python
cantod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
cantod status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u cantod -f -o cat
```
### Check Balance
```python
cantod query bank balances canto...address1yjgn7z09ua9vms259j
```


