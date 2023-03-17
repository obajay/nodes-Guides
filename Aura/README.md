# Aura Mainnet guide

![Aura](https://user-images.githubusercontent.com/44331529/180595364-72b306db-c60b-463e-877c-57ee5acc126e.png)
![aaura](https://user-images.githubusercontent.com/44331529/180595514-1dfc72a9-b72e-477b-ab5b-54f8a5071c7d.png)


[EXPLORER](https://explorer.stavr.tech/aura-mainnet)
=

- **Minimum hardware requirements**:

| Node Type  | RAM  | Storage  | 
|------------|------|----------|
| Mainnet    | 16GB | 500GB    |

# 1) Auto_install script
```python
wget -O auram https://raw.githubusercontent.com/obajay/nodes-Guides/main/Aura/auram && chmod +x auram && ./auram
```

# 2) Manual instruction
### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 19 (one command)
```pyton
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
# Build 17.03.23
```python
cd $HOME
git clone --branch aura_v0.4.3 https://github.com/aura-nw/aura
cd aura
make
```

*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`aurad version --long | head`
+ version: Aura_v0.4.3
+ commit: 019eacad3805a0c5101904035bbbf13deed68b05

```python
aurad init STAVRguide --chain-id xstaxy-1
aurad config chain-id xstaxy-1
```

## Create/recover wallet
```python
aurad keys add <walletname>
  OR
aurad keys add <walletname> --recover
```

## Genesis
```python
wget -O $HOME/.aura/config/genesis.json "https://raw.githubusercontent.com/aura-nw/mainnet-artifacts/main/xstaxy-1/genesis.json"
```
`sha256sum ~/.aura/config/genesis.json`
+ 319943868db0397ee3b368a673cbf8758554fca879c84745bdad3e452af26696

## Download addrbook
```python
wget -O $HOME/.aura/config/addrbook.json "SOOOON"
```

## Minimum gas price/Peers/Seeds
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001uaura\"/;" ~/.aura/config/app.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.aura/config/config.toml
peers="a1f949c765bfc493ddd2e0e8477170bcc3b86a57@194.163.179.176:26656,ad1febeb65726dd3f7c4083aac558ebda85a74ed@172.31.94.13:26656,a58b4dec687b60ba05cf9a3e4cd1181b09c0661f@65.109.93.152:26656,9cbe9c1f3e782ead1395c2bc205a9757aea038ea@192.168.3.113:26656,c25f51f6506d760c16f0be044f6165ae9038faa0@136.243.95.80:26656,3e05f2b0fdd750511dbff9d3f6a47d3bc3d4b1f0@141.95.204.81:26656,65b12c86e329744de6ec109ac87cce1624f3c5fc@162.19.171.125:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.aura/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.aura/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.aura/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.aura/config/config.toml
```

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.aura/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.aura/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.aura/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.aura/config/app.toml
```
### Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.aura/config/config.toml
```
## State Sync
```python
SOOOON
```
# SnapShot (~0.4 GB) updated every 5 hours
```python
SOOON
```

# Create a service file
```python
sudo tee /etc/systemd/system/aurad.service > /dev/null <<EOF
[Unit]
Description=aurad
After=network-online.target

[Service]
User=$USER
ExecStart=$(which aurad) start
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
sudo systemctl enable aurad &&
sudo systemctl restart aurad && sudo journalctl -u aurad -f -o cat
```

## Create validator
```python
aurad tx staking create-validator \
--amount 1000000ueaura \
--from <walletName> \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.2" \
--commission-rate "0.05" \
--min-self-delegation "1" \
--details="" \
--identity="" \
--pubkey  $(aurad tendermint show-validator) \
--moniker STAVRguide \
--fees 555uaura \
--chain-id xstaxy-1 -y
```

## Delete node
```python
udo systemctl stop aurad && \
sudo systemctl disable aurad && \
rm /etc/systemd/system/aurad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .aura && \
rm -rf aura && \
rm -rf $(which aurad)
```
