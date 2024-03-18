<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 3758498.668 | 0.00044613 | 1676.77901096 |

<!-- END_TABLE -->

































































































































[🔥OUR VALIDATOR🔥](https://restake.app/beezee/bzevaloper16zk776px8ef00hmd59vgnueegyrkk3lja0nhy4)
=


# BeeZee Mainnet guide
![Beezee](https://user-images.githubusercontent.com/44331529/180596395-845e85eb-ed01-4bca-ae94-90bdbfd6e5be.png)

[EXPLORER 1](https://explorer.stavr.tech/BeeZee/staking) \
[EXPLORER 2](https://explorer.thesilverfox.pro/beezee/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 160GB    |

# 1) Auto_install script
```python
wget -O beezzeed https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/BeeZee/beezzeed && chmod +x beezzeed && ./beezzeed
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 1.20.5 (one command)
```python
ver="1.20.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
 ```
## Build (21.11.23)
```python
git clone https://github.com/bze-alphateam/bze
cd bze
git checkout v6.1.0
make install
```
*******🟢UPDATE🟢******* 21.11.23
```python
cd $HOME/bze
git fetch --all
git checkout v6.1.0
make install
bzed version --long
systemctl restart bzed && journalctl -u bzed -f -o cat
```

`bzed version --long`
+ version: 6.1.0
+ commit: e8157ad95fffb2c7429b7f5619dea888a36bcdb0
```python
bzed init STAVR_guide --chain-id beezee-1
bzed config chain-id beezee-1
```    
## Create/recover wallet
```python
bzed keys add <walletname>
   OR
bzed keys add <walletname> --recover
```

### when creating, do not forget to write down the seed phrase

# Genesis
```python
wget https://raw.githubusercontent.com/bze-alphateam/bze/main/genesis.json -O $HOME/.bze/config/genesis.json
```
## Seeds and peers
```python
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.bze/config/config.toml
seeds="6385d5fb198e3a793498019bb8917973325e5eb7@51.15.228.169:26656,ce25088267cef31f3be1ec03263524764c5c80bb@163.172.130.162:26656,102d28592757192ccf709e7fbb08e7dd8721feb1@51.15.138.216:26656,f238198a75e886a21cd0522b6b06aa019b9e182e@51.15.55.142:26656,2624d40b8861415e004d4532bb7d8d90dd0e6e66@51.15.115.192:26656,d36f2bc75b0e7c28f6cd3cbd5bd50dc7ed8a0d11@38.242.227.150:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.bze/config/config.toml
```
## Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.bze/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.bze/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.bze/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.bze/config/app.toml
```
## Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.bze/config/config.toml
```
## Download addrbook
```python
wget -O $HOME/.bze/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/BeeZee/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/bzed.service > /dev/null <<EOF
[Unit]
Description=BeeZee mainnet
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bzed) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# Snaphot 24.08.22 (0.1 GB) block height --> 2293057 (disable)
```python
# install the node as standard, but do not launch. Then we delete the .data directory and create an empty directory
rm -rf $HOME/.bze/data/
mkdir $HOME/.bze/data/

# download archive
cd $HOME
wget http://116.202.236.115:7001/bzedata.tar.gz

# unpack the archive
tar -C $HOME/ -zxvf bzedata.tar.gz --strip-components 1
# !! IMPORTANT POINT. If the validator was created earlier. Need to reset priv_validator_state.json  !!
wget -O $HOME/.bze/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/priv_validator_state.json"
cd && cat .bze/data/priv_validator_state.json
{
  "height": "0",
  "round": 0,
  "step": 0
}

# after unpacking, run the node
# don't forget to delete the archive to save space
cd $HOME
rm bzedata.tar.gz
sudo systemctl restart bzed && sudo journalctl -u bzed -f -o cat
```
        
# Start
```python
sudo systemctl daemon-reload
sudo systemctl enable bzed
sudo systemctl restart bzed
sudo journalctl -u bzed -f -o cat
```
## Create validator
```python
bzed tx staking create-validator \
--amount=1000000ubze \
--pubkey=$(bzed tendermint show-validator) \
--moniker="<moniker>" \
--identity="" \
--details="" \
--website="" \
--chain-id="beezee-1" \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1" \
--fees 500ubze \
--from=<walletname> -y
```

# Delete node
```python
sudo systemctl stop bzed
sudo systemctl disable bzed
rm /etc/systemd/system/bzed.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .bze
rm -rf bze
rm -rf $(which bzed)
```
