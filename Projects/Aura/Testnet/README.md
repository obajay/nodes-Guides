<!-- START_TABLE -->
| Trusted Delegations | Token price | Result in USD |
|-------------|---------|---------------|
| 2031504.303 | 0.04150458 | 84316.73289155 |

<!-- END_TABLE -->

# Aura testnet guide

![Aura](https://user-images.githubusercontent.com/44331529/180595364-72b306db-c60b-463e-877c-57ee5acc126e.png)
![aaura](https://user-images.githubusercontent.com/44331529/180595514-1dfc72a9-b72e-477b-ab5b-54f8a5071c7d.png)

[EXPLORER](https://explorer.stavr.tech/Aura-Testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type  | RAM  | Storage  | 
|------------|------|----------|
| Validator  | 16GB | 500GB    |

# 1) Auto_install script
```python
wget -O aurat https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Aura/Testnet/aurat && chmod +x aurat && ./aurat
```

# 2) Manual instruction
### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 19 (one command)
```pyton
ver="1.19" &&
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" &&
sudo rm -rf /usr/local/go &&
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" &&
rm "go$ver.linux-amd64.tar.gz" &&
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile &&
source $HOME/.bash_profile &&
go version
```
# Build 07.02.24
```python
cd $HOME
git clone https://github.com/aura-nw/aura && cd aura
git checkout v0.7.3-euphoria
make install
```

*******🟢UPDATE🟢******* 07.02.24
```python
cd $HOME/aura
git fetch --all
git checkout v0.7.3-euphoria
make install
aurad version --long
#version: v0.7.3-euphoria
#commit: 210c446899494ff34552fd4d0c7041c703c1ccbd
sudo systemctl restart aurad && journalctl -u aurad -f -o cat
```

`aurad version --long | grep -e commit -e version`
+ version: v0.7.3-euphoria
+ commit: 210c446899494ff34552fd4d0c7041c703c1ccbd

```python
aurad init STAVR_guide --chain-id euphoria-2
```

## Create/recover wallet
```python
aurad keys add <walletname>
    OR
aurad keys add <walletname> --recover
```
## Genesis
```python
wget https://github.com/aura-nw/testnets/raw/main/euphoria-2/euphoria-2-genesis.tar.gz
tar -xzvf euphoria-2-genesis.tar.gz
mv euphoria-2-genesis.json $HOME/.aura/config/genesis.json
```
`sha256sum ~/.aura/config/genesis.json`
+ b0ee9ed933ac5c24697637bc56335136211e8d26962b1f8622c626c90772b0d6

## Download addrbook
```python
wget -O $HOME/.aura/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Aura/Testnet/addrbook.json"
```

## Minimum gas price/Peers/Seeds
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ueaura\"/;" ~/.aura/config/app.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.aura/config/config.toml
peers="7cad1bcb2ad777dba21840832341f2ce14bae1a5@5.75.174.126:26656,705e3c2b2b554586976ed88bb27f68e4c4176a33@13.250.223.114:26656,b9243524f659f2ff56691a4b2919c3060b2bb824@13.214.5.1:26656,d334e2b9dd84346ea532ff3d43f3f7c4946845c9@144.91.122.166:26656,b91ee5c72905bc49beed2720bb882c923c68fbc9@65.108.142.47:21656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.aura/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.aura/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.aura/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.aura/config/config.toml
```

### Pruning (optional)
```python
pruning="custom" &&
pruning_keep_recent="100" &&
pruning_keep_every="0" &&
pruning_interval="10" &&
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.aura/config/app.toml &&
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.aura/config/app.toml &&
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.aura/config/app.toml &&
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.aura/config/app.toml
```
### Indexer (optional)
```python
indexer="null" &&
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.aura/config/config.toml
```
## State Sync
```python
SOON
```
# SnapShot (~0.4 GB) updated every 5 hours
```python
SOON
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
sudo systemctl daemon-reload && sudo systemctl enable aurad
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
--moniker STAVR_guide \
--fees 555ueaura \
--chain-id euphoria-2 -y
```

## Delete node
```python
sudo systemctl stop aurad
sudo systemctl disable aurad
rm /etc/systemd/system/aurad.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .aura
rm -rf aura
rm -rf $(which aurad)
```
