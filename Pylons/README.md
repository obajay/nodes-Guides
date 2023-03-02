# Guide Pylons Mainnet
![pylo](https://user-images.githubusercontent.com/44331529/182419013-c3e5e07d-08de-4459-aa1c-88af51d6f340.png)

[Website](https://www.pylons.tech/home/)
=
[EXPLORER 1](https://explorer.stavr.tech/pylons/staking) \
[EXPLORER 2](https://pylons.explorers.guru/validators) 
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 200GB    |
### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y

## GO 19.5 (one command)
```pytohn
ver="1.19.5" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Binary   02.03.23
```python 
cd $HOME
git clone https://github.com/Pylons-tech/pylons
cd pylons
git checkout v1.1.4
make build && make install
```

*******🟢UPDATE🟢******* 02.03.23

```python
cd $HOME/pylons
git fetch --all
git checkout v1.1.4
make install
pylonsd version --long | head
sudo systemctl restart pylonsd && journalctl -u pylonsd -f -o cat
```

`pylonsd version version --long`
+ version: 1.1.4
+ commit: 

## Initialisation
```python
pylonsd init STAVRguide --chain-id=pylons-mainnet-1
```
## Add wallet
```python
pylonsd keys add <walletName>
          OR
pylonsd keys add <walletName> --recover
```
# Genesis
```python
curl -s  https://raw.githubusercontent.com/Pylons-tech/pylons/main/networks/pylons-mainnet-1/genesis.json > ~/.pylons/config/genesis.json
```

`sha256sum $HOME/.pylons/config/genesis.json`
- c6e776a1de29a57ce4cb4d2cdbaa39ba5768e066fd4f097cca92ce7fcfa94a9c

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.pylons/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.pylons/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.pylons/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.pylons/config/app.toml
```

### Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.pylons/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ubedrock\"/;" ~/.pylons/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.pylons/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.pylons/config/config.toml

peers="2c50b8171af784f1dca3d37d5dda5e90f1e1add8@95.214.55.4:26656,4f90babf520599ffe606157b0151c4c9bc0ec23f@194.163.172.115:26666,ebecc93e7865036fbdf8d3d54a624941d6e41ba1@104.200.136.57:26656,25e7ef64b41a636e3fb4e9bb1191b785e7d1d5cc@46.166.140.172:26656,2c50b8171af784f1dca3d37d5dda5e90f1e1add8@95.214.55.4:26656,4f90babf520599ffe606157b0151c4c9bc0ec23f@194.163.172.115:26666,ebecc93e7865036fbdf8d3d54a624941d6e41ba1@104.200.136.57:26656,022ee5a5231a5dec014841394f8ce766d657cff5@95.214.53.132:26156,a6972be573807d34f28a337c0f7d599e0014be80@161.97.99.247:26656,515ffd755a92a47b56233143f7c25481dbe99f94@161.97.99.251:26606,9c3261f7859a4f43a72cb9eef8d1fcfc70dc7e7c@95.216.204.255:26656,f6a9cc00142a4ce2fc1cbe536ba7ac9701f0786f@62.113.119.213:11221,665a747edcb6c68d3fe317053bd2cbcae1ef0843@138.201.246.185:26656,6144bf581d89212bf294de31e66f94d628f09053@65.109.92.235:37656,cbad56deb74e1349e5ed8d00cd1338c675d71075@167.86.75.50:26656,84350ef836590a08e273253f1056eb7175f536a4@167.235.2.206:26616,71b2ccc335a2ed88854444d23c2f2e2fd343c7e9@65.109.52.156:9656,85e236a129337efe946c6a68ee72a6da87825bc5@65.109.92.240:20256,3336e645081fcddb72917c017ae232fa6b7c8cf4@84.46.248.174:26656,e55c36e7ce120599701b14532c864bec57d4477b@161.97.132.66:26656,d977d11f5741d8e9be84faa390af55de43659f0c@95.217.225.214:28656,d0769a0e7fa1fc86baa0b2b9e9c6d9f7ba2dd2b6@46.4.23.108:36656,5eb3daf435d1d8a14e0a42e9dfbeca6877b2d1ca@65.108.2.41:46656,90e9144c74d83f966fbbda20c070a28d3d6e48a2@65.108.135.211:46656,7b6b13bcbd30311a407e193d0c7b21ed3dc15cd1@pylons.nodejumper.io:30656,,d6685eb44553000f5e7abfd560a7c70b534dcc25@65.108.199.222:21116,f8f74d840f40168111353927e91f475a262d20ad@65.21.142.30:27656,98634f7f77334b0df7b9c4d16d41b31ace4ceaa8@81.16.237.142:11223,d71cb7a9cc84e3c06ce2dc90f340d21ae53390ff@54.37.129.164:46656,35c6b3b3f273e845da511751d98b54ca3fd56170@65.109.49.163:26651,574a9497ef09f0364a7623ca45d7a5a067f4bc40@64.227.144.199:26656,2a21d71e5e16222fa08454634cad5db30d56dfa9@192.248.159.61:26656,1233d3696f3fbfb44edfaf72640bb91d085f3dae@65.109.34.133:61356,0d876a9311613a716a65f588c86c87f47e321945@pylons.sergo.dev:12213,32cf1fa54cb6762ea2713d93bd76c47ad3323d1c@88.99.243.241:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.pylons/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.pylons/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.pylons/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.pylons/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.pylons/config/addrbook.json "SOON"
```


# Create a service file
```python
sudo tee /etc/systemd/system/pylonsd.service > /dev/null <<EOF
[Unit]
Description=Pylons
After=network-online.target

[Service]
User=$USER
ExecStart=$(which pylonsd) start
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
sudo systemctl enable pylonsd &&
sudo systemctl restart pylonsd && sudo journalctl -u pylonsd -f -o cat
```

## Create validator
```python
pylonsd tx staking create-validator \
--amount 1000000ubedrock \
--from <walletname> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(pylonsd tendermint show-validator) \
--moniker STAVRguide \
--chain-id pylons-mainnet-1 \
--gas="auto" \
--fees 100ubedrock
-y
```

### Delete node (one command)
```python
sudo systemctl stop pylonsd && \
sudo systemctl disable pylonsd && \
rm /etc/systemd/system/pylonsd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .pylons && \
rm -rf pylons && \
rm -rf $(which pylonsd)
```
