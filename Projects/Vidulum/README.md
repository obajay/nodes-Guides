# Vidulum Mainnet guide

![vidulum](https://github.com/obajay/nodes-Guides/assets/44331529/b9a11d68-57da-47db-88ab-95f21593ca54)


[WebSite](https://vidulum.app/)\
[GitHub](https://github.com/vidulum)
=
[EXPLORER](https://explorer.stavr.tech/Vidulum-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O vdlm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Vidulum/vdlm && chmod +x vdlm && ./vdlm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.19
```python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 18.10.22
```python
cd $HOME
git clone https://github.com/vidulum/mainnet vidulum
cd vidulum
git checkout v1.2.0
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`vidulumd version --long | head`
- version: 1.2.0
- commit: latest

```python
vidulumd init STAVRguide --chain-id vidulum-1
vidulumd config chain-id vidulum-1
```    

## Create/recover wallet
```python
vidulumd keys add <walletname>
            OR
vidulumd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.vidulum/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Vidulum/genesis.json"

```
`sha256sum $HOME/.vidulum/config/genesis.json`
+ fa4c12c5c7e796a3804960cd502edb8a3171f3c5dd421a1ce69b96351cd785a3

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001uvdl\"/;" ~/.vidulum/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.vidulum/config/config.toml
peers="b63833b8b8740660ae3ac87f058447465f91f8f4@65.109.28.177:26726"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.vidulum/config/config.toml
seeds="883ec7d5af7222c206674c20c997ccc5c242b38b@ec2-3-82-120-39.compute-1.amazonaws.com:26656,eed11fff15b1eca8016c6a0194d86e4a60a65f9b@apollo.erialos.me:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.vidulum/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.vidulum/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.vidulum/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.vidulum/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.vidulum/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.vidulum/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.vidulum/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.vidulum/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.vidulum/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Vidulum/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/vidulumd.service > /dev/null <<EOF
[Unit]
Description=vidulumd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which vidulumd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Vidulum Mainnet
```python
SOOON
```
# SnapShot Mainnet (~3GB) updated every 7 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable vidulumd
sudo systemctl restart vidulumd && sudo journalctl -u vidulumd -f -o cat
```

### Create validator
```python
vidulumd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount=1000000uvdl \
--pubkey $(vidulumd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id vidulum-1 \
--fees="20000uvdl" \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop vidulumd
sudo systemctl disable vidulumd
rm /etc/systemd/system/vidulumd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf vidulum
rm -rf .vidulum
rm -rf $(which vidulumd)
```
#
### Sync Info
```python
vidulumd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
vidulumd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u vidulumd -f -o cat
```
### Check Balance
```python
vidulumd query bank balances vdl...addressjkl1yjgn7z09ua9vms259j
```
