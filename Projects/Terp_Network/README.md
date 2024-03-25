[🔥OUR VALIDATOR🔥](https://restake.app/terpnetwork/terpvaloper1eff25w2su9zxhe9lzea65l9xyptv8saxj6r2c8)
=

# Terp Mainnet guide
![terp](https://user-images.githubusercontent.com/44331529/232221589-beeb5e24-82b8-4eaf-ad51-9e3c7fecd79b.png)

[WebSite](https://terp.network/)\
[GitHub](https://github.com/terpnetwork/terp-core.git)
=
[EXPLORER](https://explorer.stavr.tech/Terp-Mainnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB   |


# 1) Auto_install script
```python
wget -O terpm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Terp_Network/terpm && chmod +x terpm && ./terpm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.21.4
```python
ver="1.21.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 26.03.24
```python
cd $HOME
git clone https://github.com/terpnetwork/terp-core.git
cd terp-core
git checkout v4.2.0
make install
```
*******🟢UPDATE🟢******* 26.03.24
```python
cd $HOME/terp-core
git fetch --all
git checkout v4.2.0
make install
terpd version --long | grep -e commit -e version
#commit: dfb1cbd64be3dd7e59320a2b7997608eacb6a5c0
#version: 4.2.0
sudo systemctl restart terpd && journalctl -u terpd -f -o cat
```

`terpd version --long | grep -e commit -e version`
- version: 4.2.0
- commit: dfb1cbd64be3dd7e59320a2b7997608eacb6a5c0

```python
terpd init STAVRguide --chain-id morocco-1
terpd config chain-id morocco-1
```    

## Create/recover wallet
```python
terpd keys add <walletname>
  OR
terpd keys add <walletname> --recover
```

## Download Genesis
```python
curl -Ls https://snapshots.nodestake.top/terp/genesis.json > $HOME/.terp/config/genesis.json 

```
`sha256sum $HOME/.terp/config/genesis.json`
+ ab6c68c50d45cb9a145edf6b37c05cb9eefc2a0488d08498b8f827c2471ba843

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uterp\"/;" ~/.terp/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.terp/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.terp/config/config.toml
peers="5c4d3ee03d080b3cb21a0b585e09da7ef56a82f3@192.81.208.147:26656,f9b67e231c59b480e1f1f9ce158f166a4b9ee829@162.19.238.161:26656,da9a83ef835387e3813bd5cd79b1b0193f522d7c@65.21.152.68:26656,439f7a680cc645d888317cd64f9b8a6949de394b@65.109.154.185:26656,297d9cf62f4414cf20c3b4150ccc7b0583ea311b@185.144.99.18:36656,c71e63b5da517984d55d36d00dc0dc2413d0ce03@143.110.219.177:26656,ed791e0800539a51efd07cfdef1f3a6809412bc1@65.109.174.30:64656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.terp/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.terp/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.terp/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.terp/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.terp/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.terp/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.terp/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.terp/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.terp/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.terp/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Terp_Network/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/terpd.service > /dev/null <<EOF
[Unit]
Description=terp
After=network-online.target

[Service]
User=$USER
ExecStart=$(which terpd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Terp Mainnet
```python
SOON
```
# SnapShot Mainnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable terpd
sudo systemctl restart terpd && sudo journalctl -u terpd -f -o cat
```

### Create validator
```python
terpd tx staking create-validator \
  --amount="1000000"uterp \
  --pubkey=$(terpd tendermint show-validator) \
  --moniker="STAVRguide" \
  --details="" \
  --website="" \
  --identity "" \
  --chain-id="morocco-1" \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.1" \
  --min-self-delegation="1" \
  --from="wallet" -y
```

## Delete node
```python
sudo systemctl stop terpd
sudo systemctl disable terpd
rm /etc/systemd/system/terpd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf terp-core
rm -rf .terp
rm -rf $(which terpd)
```
#
### Sync Info
```python
terpd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
terpd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u terpd -f -o cat
```
### Check Balance
```python
terpd query bank balances terp...addressjkl1yjgn7z09ua9vms259j
```
