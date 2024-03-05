<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 68072.2 | 0.065261 | 4442.464834214850466492 |

<!-- END_TABLE -->



























[🔥OUR VALIDATOR🔥](https://restake.app/haqq/haqqvaloper1yj2hjxsu7gxcdje86hgnm9gn7z09ua9vhjx6pn)
=

# Guide Mainnet haqq
![haqq](https://user-images.githubusercontent.com/44331529/185350224-62b92bc1-bd4e-4ce7-a56b-0abfc631c95c.png)

[Website](https://islamiccoin.net/)
=
[EXPLORER 1](https://explorer.stavr.tech/HAQQ-Mainnet/staking) \
[EXPLORER 2](https://haqq.explorers.guru/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 200GB    |

# 1) Auto_install script
```python
wget -O haqqm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/haqq/haqqm && chmod +x haqqm && ./haqqm
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
```

## GO 1.20.3 (one command) 
```
ver="1.20.3" &&
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" &&
sudo rm -rf /usr/local/go &&
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" &&
rm "go$ver.linux-amd64.tar.gz" &&
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile &&
source $HOME/.bash_profile &&
go version
```

# Binary   22.02.24
```python
cd $HOME
wget https://github.com/haqq-network/haqq/releases/download/v1.7.3/haqq_1.7.3_Linux_x86_64.tar.gz
tar -xvzf haqq_1.7.3_Linux_x86_64.tar.gz
cd bin && chmod +x haqqd
cd bin
chmod +x haqqd
mv haqqd $HOME/go/bin/
cd $HOME && rm -rf haqq_1.7.3_Linux_x86_64.tar.gz
```
*******🟢UPDATE🟢******* 22.02.24
```python
cd $HOME
wget https://github.com/haqq-network/haqq/releases/download/v1.7.3/haqq_1.7.3_Linux_x86_64.tar.gz
tar -xvzf haqq_1.7.3_Linux_x86_64.tar.gz
cd bin && chmod +x haqqd
mv haqqd $(which haqqd)
cd $HOME && rm -rf haqq_1.7.3_Linux_x86_64.tar.gz
haqqd version --long | grep -e commit -e version
#version: 1.7.2
#commit: 1e0f366008f31688952720c98eebdc552d1a6d72
sudo systemctl restart haqqd && journalctl -u haqqd -f -o cat
```
`haqqd version --long | grep -e version -e commit`
- version: 1.7.3
- commit: a4acbbe8b771e6d0ad36040197558d7ff30179b2

## Initialisation
```python
haqqd init STAVR_guide --chain-id=haqq_11235-1
haqqd config chain-id haqq_11235-1
```
## Add wallet
```python
haqqd keys add <walletName>
        OR
haqqd keys add <walletName> --recover
```
# Genesis
```python
wget -O $HOME/.haqqd/config/genesis.json "https://raw.githubusercontent.com/haqq-network/mainnet/master/genesis.json"

```

`sha256sum $HOME/.haqqd/config/genesis.json`
- e381ec1785b8d53db036a54d5a4374a83530a1083116cdec568a1123afd0f8b1  genesis.json

### Pruning
```python
pruning="custom" && \
pruning_keep_recent="10000" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.haqqd/config/app.toml
```
### Indexer (optional) one command
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.haqqd/config/config.toml

### Set up the minimum gas price and Peers/Seeds/Filter peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025aISLM\"/;" ~/.haqqd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.haqqd/config/config.toml
seeds=""
peers=""
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.haqqd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.haqqd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.haqqd/config/config.toml
```

# SnapShot
```python
SOOOON
```

## Download addrbook
```python
wget -O $HOME/.haqqd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/haqq/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/haqqd.service > /dev/null <<EOF
[Unit]
Description=haqqd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which haqqd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```


# Start node (one command)
```python
sudo systemctl daemon-reload &&
sudo systemctl enable haqqd &&
sudo systemctl restart haqqd && sudo journalctl -u haqqd -f -o cat
```

## Create validator
```python
haqqd tx staking create-validator \
--amount 1000000000000000000aISLM \
--from wallet \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.20" \
--commission-rate "0.05" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(haqqd tendermint show-validator) \
--moniker "STAVR_Guide" \
--chain-id haqq_11235-1 \
--gas 300000 -y
```

### Delete node (one command)
```python
sudo systemctl stop haqqd
sudo systemctl disable haqqd
rm /etc/systemd/system/haqqd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .haqqd
rm -rf haqq
rm -rf $(which haqqd)
```
#
### Sync Info
```python
source $HOME/.bash_profile
haqqd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
haqqd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u haqqd -f -o cat
```
### Check Balance
```python
haqqd query bank balances haqq...addresshaqq1yjgn7z09ua9vms259j
```
