# Kyve testnet guide kyve-kaon-1
![kyve](https://user-images.githubusercontent.com/44331529/224904387-84e58953-38ac-400d-b497-27172a9c8641.png)


[GitHub](https://github.com/KYVENetwork/networks/tree/main/kaon-1)
=

[EXPLORER](https://explorer.stavr.tech/kyve-kaon/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| kyve-kaon |   4| 8GB  | 160GB    |

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 19.4 (one command)
```python
cd $HOME && \
ver="1.19.4" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 09.03.23
```python
cd $HOME
wget https://github.com/KYVENetwork/chain/releases/download/v1.0.0-rc1/kyved_linux_amd64.tar.gz
tar -xvzf kyved_linux_amd64.tar.gz
chmod +x ./kyved
mv kyved $HOME/go/bin/
```

*******🟢UPDATE🟢******* 09.03.23
```python
cd $HOME
wget https://github.com/KYVENetwork/chain/releases/download/v1.0.0-rc1/kyved_linux_amd64.tar.gz
tar -xvzf kyved_linux_amd64.tar.gz
chmod +x ./kyved
mv kyved $(which kyved)
kyved version --long
#version: v1.0.0-rc1
#commit: cedc217027c2b7e3d2b7077abe7a4fa4442a19c4
systemctl restart kyved && journalctl -u kyved -f -o cat
```
`kyved version --long`
+ version: v1.0.0-rc1
+ commit: cedc217027c2b7e3d2b7077abe7a4fa4442a19c4


```python
kyved init STAVRguide --chain-id kaon-1
kyved config chain-id kaon-1
```

## Create/recover wallet
```python
kyved keys add <walletname>
      or
kyved keys add <walletname> --recover
```

## Genesis
```python
wget -O $HOME/.kyve/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Kyve/Kaon/genesis.json"
```

## Peers/Seeds/MaxPeers/FilterPeers
```python
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.kyve/config/config.toml
seeds=""
peers="f5f83485ce4fc708dfa8b4de22361fdd15fba3ee@192.168.0.97:26656,ed9989e0b003b24f3d38d060017b73af5c61b18c@192.168.1.118:26656,78d76da232b5a9a5648baa20b7bd95d7c7b9d249@142.93.161.118:26656,61909d4ad9fac1890d69b93612e7a4177c8d1104@192.168.1.177:26656,aaa8a6f7eab9d20e87bcc01ddd53616cbd203c36@136.243.88.91:26656"
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
```python
ndexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.kyve/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.kyve/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Kyve/Kaon/addrbook.json"
```



# Create a service file
```python
sudo tee <<EOF > /dev/null /etc/systemd/system/kyved.service
[Unit]
Description=KYVE
After=network-online.target

[Service]
User=$USER
ExecStart=$(which kyved) start
Restart=on-failure
RestartSec=10
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload && sudo systemctl enable kyved && 
sudo systemctl restart kyved && sudo journalctl -u kyved -f -o cat
```
## Create validator
```python
kyved tx staking create-validator \
-amount 10000000000tkyve \
--moniker="STAVRGuide" \
--identity="<identity>" \
--website="<website>" \
--details="<any details>" \
--commission-rate "0.10" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.05" \
--min-self-delegation "1" \
--pubkey "$(kyved tendermint show-validator)" \
--from <your-key-name> \
--chain-id kaon-1
```

## Delete node
```python
sudo systemctl stop kyved && \
sudo systemctl disable kyved && \
rm /etc/systemd/system/kyved.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
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
