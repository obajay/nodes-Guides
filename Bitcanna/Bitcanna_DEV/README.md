# Bitcanna Devnet guide


<img src='https://user-images.githubusercontent.com/44331529/190848563-484255a3-60cc-4259-a6ba-e49adf48ebeb.gif' alt='Bitcanna'  width='50%'>

[<img src='https://user-images.githubusercontent.com/44331529/190849033-ceade049-eb11-47de-93a0-38d96caed8b8.png' alt='Bitcanna'  width='100%'>](https://medium.com/@BitCannaGlobal/introducing-bitcanna-buddheads-nft-45f2e05fd191)



[Website](https://www.bitcanna.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/bitcanna-dev/staking) \
[EXPLORER 2](https://testnets-cosmos.mintthemoon.xyz/bitcanna/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4|  8GB | 260GB    |


### Preparing the server
```bash
udo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 18.5

```bash
cd $HOME
ver="1.18.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```


### 1_Previously to replace the binary or build from source you need to edit the file app.toml
#### you should to declare and put in false this var: `iavl-disable-fastnode = false`
#### Put this content in the main section, just before the [telemetry] section:
```bash
# IavlCacheSize set the size of the iavl tree cache. 
# Default cache size is 50mb.
iavl-cache-size = 781250

# IAVLDisableFastNode enables or disables the fast node feature of IAVL. 
# Default is true.
iavl-disable-fastnode = false  
###############################################################################
###                         Telemetry Configuration                         ###
###############################################################################
```


# Build 07.11.22
```python
cd ~
wget -nc https://github.com/BitCannaGlobal/bcna/releases/download/v2.0.1-rc2/bcna_linux_amd64.tar.gz
tar zxvf  bcna_linux_amd64.tar.gz
rm bcna_linux_amd64.tar.gz
sudo mv bcnad $HOME/go/bin/
```

`bcnad version`
- 2.0.1-rc2

```bash
bcnad init STAVRguide --chain-id bitcanna-dev-5
```    

## Create/recover wallet
```bash
bcnad keys add <walletname>
bcnad keys add <walletname> --recover
```

## Download Genesis

```bash
wget -O $HOME/.bcna/config/genesis.json "https://raw.githubusercontent.com/BitCannaGlobal/testnet-bcna-cosmos/main/instructions/bitcanna-dev-5/genesis.json"
```
`sha256sum $HOME/.bcna/config/genesis.json`
+ 2c4aeec0d2b416478de0d6bd5f32954f18235ae1991e9f1b685946963e3ca425

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0stake\"/;" ~/.bcna/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.bcna/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.bcna/config/config.toml
peers="3912bbbe1beeab27c5050558d625ac6334b1e5e8@212.227.151.143:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bcna/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.bcna/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.bcna/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.bcna/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.bcna/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.bcna/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.bcna/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.bcna/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.bcna/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.bcna/config/addrbook.json "https://github.com/obajay/nodes-Guides/edit/main/Bitcanna/Bitcanna_DEV/addrbook.json"
```


# StateSync
```python
RPC="https://test-rpc-bitcanna.mintthemoon.xyz:443"
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.bcna/config/config.toml
bcnad tendermint unsafe-reset-all --home /root/.bcna --keep-addr-book
sudo systemctl restart bcnad && sudo journalctl -u bcnad -f -o cat
```

# Create a service file
```bash
sudo tee /etc/systemd/system/bcnad.service > /dev/null <<EOF
[Unit]
Description=bitcanna
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bcnad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```bash
sudo systemctl daemon-reload && sudo systemctl enable bcnad
sudo systemctl restart bcnad && sudo journalctl -u bcnad -f -o cat
```

### Create validator
```bash
bcnad tx staking create-validator \
--amount=1000000ubcna \
--broadcast-mode=block \
--pubkey=`bcnad tendermint show-validator` \
--moniker=STAVRguide \
--commission-rate="0.1" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--from=<walletName> \
--chain-id=bitcanna-dev-5 \
--gas=auto -y
```

## Delete node
```bash
sudo systemctl stop bcnad && \
sudo systemctl disable bcnad && \
rm /etc/systemd/system/bcnad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf bcna && \
rm -rf .bcna && \
rm -rf $(which bcnad)
```

