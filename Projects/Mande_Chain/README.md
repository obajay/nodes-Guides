# Mande Chain Testnet guide

![mmm](https://user-images.githubusercontent.com/44331529/195984832-4b59ffcb-4253-40ee-9168-edc7bfa7425f.png)

[WEBSITE](https://www.mande.network/) \
[GitHub](https://github.com/mande-labs/testnet-2)
=
[EXPLORER 1](https://explorer.stavr.tech/Mande-Chain/staking) \
[EXPLORER 2](https://test.anode.team/mande-network/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Tetsnet   |   2|  4GB | 100GB    |


# 1) Auto_install script
```bash
wget -O mnd https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Mande_Chain/mnd && chmod +x mnd && ./mnd
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
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

# Build 03.03.23
```python
cd $HOME
git clone https://github.com/mande-labs/mande-chain.git
cd mande-chain
git checkout v1.2.2
ignite chain build --release
 #or jus download binary file -->> https://drive.google.com/file/d/19tSlJRWKyYpLBWkWTMC6NQYOCrViE9-9/view?usp=share_link
cd $HOME/mande-chain/release/
tar -xvzf mande-chain_linux_amd64.tar.gz
mv mande-chaind $HOME/go/bin/

```
`mande-chaind version --long | head`
- version: 1.2.2
- commit: 23e306bc798d1d01ddb409ff6bce93478aa03fc4

```python
mande-chaind init STAVRguide --chain-id mande-testnet-2
mande-chaind config chain-id mande-testnet-2
```    

## Create/recover wallet
```python
mande-chaind keys add <walletname>
mande-chaind keys add <walletname> --recover
```

## Download Genesis

```python
wget -O $HOME/.mande-chain/config/genesis.json "https://raw.githubusercontent.com/mande-labs/testnet-2/main/genesis.json"
```
`sha256sum $HOME/.mande-chain/config/genesis.json`
+ 7b139aea9b1d4210ab89078bd2ec2cd28b86d10870232d07dabdf5d5472f472d

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005mand\"/;" ~/.mande-chain/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.mande-chain/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.mande-chain/config/config.toml
peers="dbd1f5b01f010b9e6ae6d9f293d2743b03482db5@34.171.132.212:26656,1d1da5742bdd281f0829124ec60033f374e9ddac@34.170.16.69:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mande-chain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.mande-chain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.mande-chain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.mande-chain/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.mande-chain/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mande-chain/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.mande-chain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Mande_Chain/addrbook.json"
```

# StateSync
```python
SOON

```

# Create a service file
```python
sudo tee /etc/systemd/system/mande-chaind.service > /dev/null <<EOF
[Unit]
Description=mande-chaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which mande-chaind) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable mande-chaind
sudo systemctl restart mande-chaind && sudo journalctl -u mande-chaind -f -o cat
```
## Use faucet
[FAUCET](https://discord.com/channels/953348696098103366/1033430536129101904)
=

### Create validator
```python
mande-chaind tx staking create-validator \
--chain-id mande-testnet-2 \
--amount 0cred \
--pubkey "$(mande-chaind tendermint show-validator)" \
--from <wallet> \
--moniker="STAVRguide" \
--fees 1000mand \
--gas-adjustment=1.15 -y
```

## Delete node
```python
sudo systemctl stop mande-chaind && \
sudo systemctl disable mande-chaind && \
rm /etc/systemd/system/mande-chaind.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .mande-chain && \
rm -rf $(which mande-chaind)
```
