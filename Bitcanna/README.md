# Bitcanna mainnet guide


<img src='https://user-images.githubusercontent.com/44331529/190848563-484255a3-60cc-4259-a6ba-e49adf48ebeb.gif' alt='Bitcanna'  width='50%'>

[<img src='https://user-images.githubusercontent.com/44331529/190849033-ceade049-eb11-47de-93a0-38d96caed8b8.png' alt='Bitcanna'  width='100%'>](https://medium.com/@BitCannaGlobal/introducing-bitcanna-buddheads-nft-45f2e05fd191)



[Website](https://www.bitcanna.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/bitcanna/staking) \
[EXPLORER 2](https://www.mintscan.io/bitcanna/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 260GB    |


# 1) Auto_install script
```bash
wget -O bitcanna https://raw.githubusercontent.com/obajay/nodes-Guides/main/Bitcanna/bitcanna && chmod +x bitcanna && ./bitcanna
```

# 2) Manual installation

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

# Build 11.10.22
```bash
cd ~
wget https://github.com/BitCannaGlobal/bcna/releases/download/v1.4.3/bcna_linux_amd64.tar.gz
tar zxvf bcna_linux_amd64.tar.gz
./bcnad version --long --output json |jq .commit 
   >>>>> output should be >>>> "94bfd1b95655df23ec5617b06aadf80f90917521"
sudo mv ./bcnad $HOME/go/bin/
rm -rf bcna_linux_amd64.tar.gz
```

`bcnad version`
- 

```bash
bcnad init STAVRguide --chain-id bitcanna-1
```    

## Create/recover wallet
```bash
bcnad keys add <walletname>
bcnad keys add <walletname> --recover
```

## Download Genesis

```bash
wget -O $HOME/.bcna/config/genesis.json "https://raw.githubusercontent.com/BitCannaGlobal/bcna/main/genesis.json"
```
`sha256sum $HOME/.bcna/config/genesis.json`
+ cd7449a199e71c400778f894abb00874badda572ac5443b7ec48bb0aad052f29

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0stake\"/;" ~/.bcna/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.bcna/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.bcna/config/config.toml
peers="0bf629f4e055af47f7c35bb444cb9013d18b9941@141.95.124.151:21326"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bcna/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.bcna/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.bcna/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.bcna/config/config.toml

```
### Pruning (optional)
```bash
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
wget -O $HOME/.bcna/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Bitcanna/addrbook.json"
```


# StateSync
```bash
RPC="http://bitcanna.rpc.m.stavr.tech:21327"
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="f5ab20ff53ddaa2a473c23694b4bac2c187d7b2a@135.181.5.47:21326"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bcna/config/config.toml
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.bcna/config/config.toml
sudo systemctl stop bcnad && bcnad tendermint unsafe-reset-all --keep-addr-book
sudo systemctl restart bcnad && sudo journalctl -u bcnad -f -o cat
```

# SnapShot 08.10.22 (0.2 GB) height 5346921
```bash
# install the node as standard, but do not launch. Then we delete the .data directory and create an empty directory
sudo systemctl stop bcnad
rm -rf $HOME/.bcna/data/
mkdir $HOME/.bcna/data/

# download archive
cd $HOME
wget http://bitcanna.snapshot.stavr.tech:5101/bitcannaindata.tar.gz

# unpack the archive
tar -C $HOME/ -zxvf bitcannaindata.tar.gz --strip-components 1
# !! IMPORTANT POINT. If the validator was created earlier. Need to reset priv_validator_state.json  !!
wget -O $HOME/.bcna/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/priv_validator_state.json"
cd && cat .bcna/data/priv_validator_state.json

# after unpacking, run the node
# don't forget to delete the archive to save space
cd $HOME
wget -O $HOME/.bcna/config/genesis.json "https://raw.githubusercontent.com/BitCannaGlobal/bcna/main/genesis.json"
rm bitcannaindata.tar.gz
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
--chain-id=bitcanna-1 \
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

