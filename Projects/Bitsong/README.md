<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 349714.065 | 0.01420694 | 4968.36674378 |

<!-- END_TABLE -->





























[🔥OUR VALIDATOR🔥](https://restake.app/bitsong/bitsongvaloper1c5p4sqgz5jslpywsk5c0nasqqjfucv9lvjlnry)
=

# Bitsong Mainnet guide
![bitsong (1)](https://user-images.githubusercontent.com/44331529/180596926-fde4ee88-930f-402c-a349-c0576bf38448.png)
![bitsong (2)](https://user-images.githubusercontent.com/44331529/180596927-397fc2a5-5d1c-4d52-9f51-a9fe800f8977.png)

[Website](https://bitsong.io/)
 =
[EXPLORER 1](https://explorer.stavr.tech/Bitsong/staking) \
[EXPLORER 2](https://www.mintscan.io/bitsong/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 260GB    |

# 1) Auto_install script 
```python
wget -O bitsongd https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Bitsong/bitsongd && chmod +x bitsongd && ./bitsongd
```
# 2) Manual installation

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

# Build 15.03.24
```python
git clone https://github.com/bitsongofficial/go-bitsong/
cd go-bitsong
git checkout v0.15.0
make install
```
🟢UPDATE🟢 15.03.24
```python
cd $HOME/go-bitsong
git fetch --all
git checkout v0.15.0
make install
bitsongd version
sudo systemctl restart bitsongd && journalctl -u bitsongd -f -o cat

```

`bitsongd version version --long | head`
- version: 0.15.0
- commit: 456826d85f68958c30bc3c731b6ed260cec1aa75

```python
bitsongd init STAVR_guide --chain-id bitsong-2b
```

## Create/recover wallet
```python
bitsongd keys add <walletname>
bitsongd keys add <walletname> --recover
```
#### when creating, do not forget to write down the seed phrase
## Genesis
```python
wget -O $HOME/.bitsongd/config/genesis.json "https://raw.githubusercontent.com/bitsongofficial/networks/master/bitsong-2b/genesis.json"
```

## Set up the minimum gas price $HOME/.bitsongd/config/app.toml as well as seed and peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001ubtsg\"/;" ~/.bitsongd/config/app.toml

external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.bitsongd/config/config.toml

peers="2af83a42fc643ddeeb5a1a10afc70fe0fa7e9201@wisdom.bonded.zone:26956"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bitsongd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.bitsongd/config/config.toml 
```

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.bitsongd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.bitsongd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.bitsongd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.bitsongd/config/app.toml
```
### Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.bitsongd/config/config.toml
```
# Download addrbook
```python
wget -O $HOME/.bitsongd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Bitsong/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/bitsongd.service > /dev/null <<EOF
[Unit]
Description=bitsong
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bitsongd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## StateSync
```python
RPC="51.195.189.48:21037"
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="eddccf92e8d7d746f6a42ad08b9800037e3633cf@51.195.189.48:21036"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bitsongd/config/config.toml

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.bitsongd/config/config.toml
bitsongd tendermint unsafe-reset-all --home /root/.bitsongd --keep-addr-book
sudo systemctl restart bitsongd && journalctl -u bitsongd -f -o cat
```

## Snaphot 26.08.22 (0.8 GB) block height --> 7450094
```python
# install the node as standard, but do not launch. Then we delete the .data directory and create an empty directory
sudo systemctl stop bitsongd
rm -rf $HOME/.bitsongd/data/
mkdir $HOME/.bitsongd/data/
# download archive
cd $HOME
wget http://51.195.189.48:7000/bitsongddata.tar.gz
# unpack the archive
tar -C $HOME/ -zxvf bitsongddata.tar.gz --strip-components 1
# !! IMPORTANT POINT. If the validator was created earlier. Need to reset priv_validator_state.json  !!
wget -O $HOME/.bitsongd/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/priv_validator_state.json"
cd && cat .bitsongd/data/priv_validator_state.json
{
  "height": "0",
  "round": 0,
  "step": 0
}
# after unpacking, run the node
# don't forget to delete the archive to save space
cd $HOME
rm bitsongddata.tar.gz
sudo systemctl restart bitsongd && sudo journalctl -u bitsongd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable bitsongd
sudo systemctl restart bitsongd
sudo journalctl -u bitsongd -f -o cat
```
## Create validator
```python
bitsongd tx staking create-validator \
--amount 1000000ubtsg \
--from <walletName> \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.2" \
--commission-rate "0.05" \
--min-self-delegation "1" \
--details="" \
--identity="" \
--pubkey  $(bitsongd tendermint show-validator) \
--moniker <moniker> \
--fees 2000ubtsg \
--chain-id bitsong-2b -y
```
## Delete node
```python
sudo systemctl stop bitsongd
sudo systemctl disable bitsongd
rm /etc/systemd/system/bitsongd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .bitsongd
rm -rf go-bitsong
rm -rf $(which bitsongd)
```
