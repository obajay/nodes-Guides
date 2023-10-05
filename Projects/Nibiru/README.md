# Nibiru testnet  guide

![nibiru](https://user-images.githubusercontent.com/44331529/199216266-6b0da979-44a2-43e4-b9ef-de3a7c361b17.png)

[Website](https://nibiru.fi/) \
[GitHub](https://github.com/NibiruChain)
=
[EXPLORER 1](https://explorer.stavr.tech/Nibiru/staking) \
[EXPLORER 2](https://exp.utsa.tech/nibiru-test/staking)
=
- **Minimum hardware requirements**:


| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4| 8GB  | 100GB    |

# 1) Auto_install script
```python 
wget -O nib https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nibiru/nib && chmod +x nib && ./nib
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
```

## GO 19 (one command) 
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

# Binary   04.10.23
```python 
cd $HOME
curl -s https://get.nibiru.fi/@v0.21.11! | bash
mv /usr/local/bin/nibid $HOME/go/bin
```

`nibid version --long | grep -e version -e commit`
+ v0.21.11
+ commit: b5fc784f8fb31628df639fcc23d80621e437f044

## Initialisation
```python
nibid init STAVRguide --chain-id=nibiru-itn-3
nibid config chain-id nibiru-itn-3

```
## Add wallet
```python
nibid keys add <walletName>
nibid keys add <walletName> --recover
```
# Genesis
```python
curl -Ls https://snapshots.kjnodes.com/nibiru-testnet/genesis.json > $HOME/.nibid/config/genesis.json
```

`sha256sum $HOME/.nibid/config/genesis.json`
- 112ca8b452ca0369e0f225b9befe26dff9cfa3879af4c7f334a7a15cb8f12619  genesis.json

### Pruning
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.nibid/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.nibid/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.nibid/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.nibid/config/app.toml
```
### Indexer (optional) one command
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.nibid/config/config.toml
```

### Set up the minimum gas price and Peers/Seeds/Filter peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025unibi\"/;" ~/.nibid/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.nibid/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.nibid/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.nibid/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.nibid/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.nibid/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.nibid/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.nibid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nibiru/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```


# Start node (one command)
```python
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl restart nibid && sudo journalctl -u nibid -f -o cat
```

## Create validator
```
nibid tx staking create-validator \
--amount=1000000unibi \
--pubkey=$(nibid tendermint show-validator) \
--moniker=STAVRguide \
--chain-id=nibiru-itn-3 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--from=<walletname> \
--identity="" \
--details="" \
--website="" \
-y
```

### Delete node (one command)
```python
sudo systemctl stop nibid
sudo systemctl disable nibid
rm /etc/systemd/system/nibid.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .nibid
rm -rf nibiru
rm -rf $(which nibid)
```
#
### Sync Info
```python
nibid status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
nibid status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u nibid -f -o cat
```
### Check Balance
```python
nibid query bank balances nibi...addressnibi1yjgn7z09ua9vms259j
```
