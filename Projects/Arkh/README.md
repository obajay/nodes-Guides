# ARKH Mainnet guide

[WebSite]( https://www.arkhadian.com/)\
[GitHub](https://github.com/vincadian/arkh-blockchain)
=
[EXPLORER](https://explorer.stavr.tech/Arkh-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O arkhm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Arkh/arkhm && chmod +x arkhm && ./arkhm
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

# Build 05.07.23
```python
cd $HOME
git clone https://github.com/vincadian/arkh-blockchain
cd arkh-blockchain
git checkout v2.0.0
go install ./...
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`arkhd version --long | head`
- version: 
- commit: 

```python
arkhd init STAVR_guide --chain-id arkh
arkhd config chain-id arkh
```    

## Create/recover wallet
```python
arkhd keys add <walletname>
  OR
arkhd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.arkh/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Arkh/genesis.json"

```
`sha256sum $HOME/.arkh/config/genesis.json`
+ cebc1ed1b7de5f82c54cb470254e5301705b2db9b505a005ecd96adabc25ac6f

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0arkh\"/;" ~/.arkh/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.arkh/config/config.toml
peers="808f01d4a7507bf7478027a08d95c575e1b5fa3c@86.247.125.164:26656,889e31730df026e6cec506e26a0791368f8073a2@162.19.236.117:26656,1af8fdecd6e8f9ec1bfcc3288fe46ce45e4df963@144.76.97.251:39656,b4f3bd0b9202be699635966978b44e5ea8ab9fba@34.173.89.239:26656,66989196aa1a32b0f120c72fd7c2ed1a5646e81a@213.239.215.77:46656,9e53ef81cba40318d52593559aa738e39132db3f@104.208.75.216:13756,ef7245f080d957b0505a8b670c37861faaf368f0@167.99.69.130:13756,1f39ccff07110aa887e0f24cee17404d41084dd9@45.151.123.72:18656,156a09b0ab5390b673c94ba7deebf15781d02563@206.189.83.42:13756,b39fec8beed5a72e77a10d213dc2c38ce9909e8b@45.141.122.178:35656,7ee7a039ea8a6fdcdb7defc0bce182d313cd68c5@65.108.97.58:25656,cc830584e010e111d75cba359475d1e9fd091139@146.190.40.38:26656,a4c02968458f3f51492bb1ee026b853e8d1c1428@46.4.75.21:13756,4835144689fb4819bada135f92c1a83b2f84c0bb@3.138.135.123:26656,35ab45eafaed0722f90622fc2c23d01739df25c5@207.154.243.48:18656,82e425a51c75e60663d310deea144a50c5206f0b@167.99.234.217:13756,1570da59051aef97736db1c16d3cf04dbe8ee7fd@194.163.167.138:56756,cccb1a885cffeb3de745996de8c3161fcd499dff@85.239.230.14:13756,9fcb65ca60bada22cfb25d46f6fc6dbe93740c26@195.201.83.242:13756,43b2a1a37fb88fa9cc3b133205633de73c807092@38.242.134.77:13756,02c049b6683c3a22ee83a1ce888c06b188bbcf6c@165.232.126.250:18656,f66afec8a1fb6c06f017a115433deee4bbf588aa@88.99.161.162:23656,9b1bda1f94e73ae71d4e3188d85da10d0a763ac2@195.3.221.58:13756,bb281b7b461cbd06dcda0220bd033207ce9b594e@213.133.103.188:13756,687dd5f8cfb62f63dc4bb04a28cfdb3225bff2e5@95.216.75.119:13756,56df4fa55c4352450cf0f7ce993f749c6b5ab2b1@34.125.177.180:13756,3c4526259c56fae4f7b4dcc17c2faacd5f8df1f3@192.249.115.155:13657,2ef71bffabed23207a95bf731e64d018d81d7877@89.116.28.131:26656,f8055e1cd617ce0fb7848cf759e540f1f06009e6@164.92.239.92:18656,71d76b7d5a90c9f289335032c3af6b1a0ecce2e9@89.117.50.187:25656,92b035580fdf4fa510d00a7bbccb107c1e611fb3@65.109.92.240:13756,9f7b574bf3a30ece3083dd6d0271d9ca617c8ddc@134.209.21.58:18656,344372a4883988741b462223790e2b28e5be1d38@81.0.218.58:13756,37f46fa598a7a419bafd936ec78d90f4fcdec8b6@87.106.112.86:13756,861ef652578905f27e7ea1f6d36b68fda08751f5@35.192.119.28:26656,7f14aaa9b8bac1b2762a7863400e427f82196976@146.190.40.115:18656,14d99058cb07ec9b14547de3b51fb8344bf24ea3@104.152.109.242:28656,0dd079e479da7c873a6da2bdd450298c10c2d51e@65.108.9.164:13756,8326d9d921afc60a2c9e7b57a48c51f7f2ae7e81@137.184.180.170:18656,ce0cde42967aa085bca9d66c0e5695d6341c778a@165.22.76.250:18656,d26c28e9c8698faea615914ffbacce81d63f072e@65.109.104.118:61456,c0cc8b6c9e42f2a2f12e2bc5b354baa51d176a66@173.212.222.167:32656,d747ddf72464065bc7d221b500b2c0e65ff34ff9@194.233.68.136:13756,f7b5d20f636fe7c2ec504662834b35b0cc56a742@194.163.165.174:37656,c9237db05716b1635df82d03cfdd498110240dcd@49.12.123.87:46656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.arkh/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.arkh/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.arkh/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.arkh/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.arkh/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.arkh/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.arkh/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.arkh/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.arkh/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.arkh/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Arkh/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/arkhd.service > /dev/null <<EOF
[Unit]
Description=ARKH
After=network-online.target

[Service]
User=$USER
ExecStart=$(which arkhd) start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Arkh Mainnet
```python
SOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable arkhd
sudo systemctl restart arkhd && sudo journalctl -u arkhd -f -o cat
```

### Create validator
```python
arkhd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.2 \
--min-self-delegation "1" \
--amount "30000000"arkh \
--pubkey $(arkhd tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id arkh \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop arkhd
sudo systemctl disable arkhd
rm /etc/systemd/system/arkhd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf arkh-blockchain
rm -rf .arkh
rm -rf $(which arkhd)
```
#
### Sync Info
```python
arkhd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
arkhd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u arkhd -f -o cat
```
### Check Balance
```python
arkhd query bank balances arkh...addressjkl1yjgn7z09ua9vms259j
```
