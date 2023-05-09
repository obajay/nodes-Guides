# Kyve Mainnet guide
![kyve](https://user-images.githubusercontent.com/44331529/224901192-a4fd240c-be4d-497b-b61f-11d8531aa13d.png)

[WebSite](https://www.kyve.network/)
=

[EXPLORER 1](https://explorer.stavr.tech/kyve/staking) \
[EXPLORER 2](https://kyve.explorers.guru/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 260GB    |

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
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
# Build 09.08.23
```python
cd $HOME
wget https://github.com/KYVENetwork/chain/releases/download/v1.1.0/kyved_mainnet_linux_amd64.tar.gz
tar -xvzf kyved_mainnet_linux_amd64.tar.gz
chmod +x kyved
rm kyved_mainnet_linux_amd64.tar.gz
sudo mv kyved $HOME/go/bin/kyved
```

`kyved version --long`
- version: v1.1.0
- commit: b07023dbbf7a2c1fc4aa5b35d4d916d17f757a91


```python    
kyved init STAVRguide --chain-id kyve-1
kyved config chain-id kyve-1
```

## Create/recover wallet
```python
kyved keys add <walletname>
kyved keys add <walletname> --recover
```
## Genesis
```python
curl https://files.kyve.network/mainnet/genesis.json > ~/.kyve/config/genesis.json
```

## Peers/Seeds/MaxPeers/FilterPeers
```python
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.kyve/config/config.toml
seeds=""
peers="b950b6b08f7a6d5c3e068fcd263802b336ffe047@18.198.182.214:26656,25da6253fc8740893277630461eb34c2e4daf545@3.76.244.30:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.kyve/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.kyve/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.kyve/config/config.toml
```

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.kyve/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.kyve/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.kyve/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.kyve/config/app.toml
```
### Indexer (optional)
```pytohn
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.kyve/config/config.toml
```
## Download addrbook
```python
wget -O $HOME/.kyve/config/addrbook.json "SOOON"
```
## State Sync
```python
SOOON
```

## SnapShot (~1 GB) updated every 5 hours
```python
SOOON
```

# Create a service file
```python
sudo tee /etc/systemd/system/kyved.service > /dev/null <<EOF
[Unit]
Description=kyve
After=network-online.target

[Service]
User=$USER
ExecStart=$(which kyved) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
## Start
```python
sudo systemctl daemon-reload && \
sudo systemctl enable kyved && \
sudo systemctl restart kyved && \
sudo journalctl -u kyved -f -o cat
```
## Create validator

```python
kyved tx staking create-validator \
--amount 10000000000ukyve \
--moniker="STAVRguide" \
--identity="<identity>" \
--website="<website>" \
--details="<any details>" \
--commission-rate "0.10" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.05" \
--min-self-delegation "1" \
--pubkey "$(kyved tendermint show-validator)" \
--from <your-key-name> \
--gas 51000000 \
--fees 1020000ukyve \
--chain-id kyve-1
```

## Delete node
```python
sudo systemctl stop kyved && \
sudo systemctl disable kyved && \
rm /etc/systemd/system/kyved.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf chain $$ \
rm -rf .kyve && \
rm -rf $(which kyved)
```

### Sync Info
```python
source $HOME/.bash_profile
kyved status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
kyved status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u kyved -f -o cat
```
### Check Balance
```python
kyved query bank balances kyve...addresshaqq1yjgn7z09ua9vms259j
```
