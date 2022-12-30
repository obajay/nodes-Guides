# Quicksilver  Testnet

![quick](https://user-images.githubusercontent.com/44331529/201520331-711f381d-89ab-4b8b-bab9-114c2b2521bd.png)


[WEBSITE](https://quicksilver.zone/)
=
[EXPLORER 1](https://explorer.stavr.tech/quicksilver/staking) \
[EXPLORER 2](https://testnet.manticore.team/quicksilver/staking)
=
### Updating the repositories  

    sudo apt update && sudo apt upgrade -y

### Installing the necessary utilities

    sudo apt install curl build-essential git wget jq make gcc tmux nvme-cli -y
    
### GO 1.19 (one command)
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
### Node installation 30.12.22
```python
cd $HOME
wget https://github.com/ingenuity-build/quicksilver/releases/download/v1.1.0-innuendo/quicksilverd-v1.1.0-innuendo-amd64
mv quicksilverd-v1.1.0-innuendo-amd64 quicksilverd
chmod +x quicksilverd
mv $HOME/quicksilverd $HOME/go/bin/
```

`quicksilverd version`
+ version: 1.1.0-innuendo


### Initialize the node
```java
quicksilverd config chain-id innuendo-4
quicksilverd init STAVRguide --chain-id innuendo-4
```
[SNAPSHOT by Kjnodes](https://services.kjnodes.com/home/testnet/quicksilver/snapshot)
=
### Create wallet or restore
    quicksilverd keys add <name_wallet>
            or
    quicksilverd keys add <name_wallet> --recover

### Download Genesis
```python
wget -O $HOME/.quicksilverd/config/genesis.json "https://raw.githubusercontent.com/ingenuity-build/testnets/main/innuendo/genesis.json"
```
`sha256sum ~/.quicksilverd/config/genesis.json`
 + 80c7d19750b6c29b8fef5de7263d6347f55681c7804300a8c207bbde0af3e6cd

### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0001uqck\"/;" ~/.quicksilverd/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.quicksilverd/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.quicksilverd/config/config.toml
peers="b9b8bb23e61d53ff3b293485d04ea567ebcd7933@65.108.65.94:26656,a94cf3e93cec8eef6d67c2972e4af5eae1a118b2@65.108.2.27:26656,926ce3f8ce4cda6f1a5ee97a937a44f59ff28fbf@65.108.13.176:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.quicksilverd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.quicksilverd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.quicksilverd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.quicksilverd/config/config.toml
```


### Setting up pruning with one command (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="50" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.quicksilverd/config/app.toml

### Disable indexing (optional)
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.quicksilverd/config/config.toml


### Download addrbook
```python
wget -O $HOME/.quicksilverd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Quicksilver/addrbook.json"
```

### Create a service file
```python
sudo tee /etc/systemd/system/quicksilverd.service > /dev/null <<EOF
[Unit]
Description=quicksilver
After=network-online.target

[Service]
User=$USER
ExecStart=$(which quicksilverd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```python    
sudo systemctl daemon-reload
sudo systemctl enable quicksilverd
sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

#### After synchronization, we go to the discord and we request coins

### Create a validator
```python
quicksilverd tx staking create-validator \
--chain-id innuendo-4 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.1 \
--min-self-delegation="1" \
--amount=1000000uqck \
--pubkey $(quicksilverd tendermint show-validator) \
--moniker "STAVRguide" \
--from=<name_wallet> \
--gas="auto" \
--fees 555uqck -y
```    

## Delete node
```python
sudo systemctl stop quicksilverd && \
sudo systemctl disable quicksilverd && \
rm /etc/systemd/system/quicksilverd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf quicksilver && \
rm -rf .quicksilverd && \
rm -rf $(which quicksilverd)
```

#

`Sync Info`
```python
quicksilverd status 2>&1 | jq .SyncInfo
```
`NodeINfo`
```python
quicksilverd status 2>&1 | jq .NodeInfo
```
`Check node logs`
```python
quicksilverd journalctl -u haqqd -f -o cat
```
`Check Balance`
```python
quicksilverd query bank balances quicksilver...addressdefund1yjgn7z09ua9vms259j
```
