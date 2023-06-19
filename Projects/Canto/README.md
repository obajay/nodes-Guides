# Guide Canto Mainnet
![canto](https://user-images.githubusercontent.com/44331529/185346490-c8f643a2-8465-432a-90dd-950b0e26957c.png)

[Website](https://canto.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/canto/staking) \
[EXPLORER 2](https://mainnet.manticore.team/canto/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB  | 260GB    |


# 1)    Auto_install script
```bash
wget -O canto https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Canto/canto && chmod +x canto && ./canto
```
# 2)    Manual installation

### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y

## GO 18.3 (one command)
```
ver="1.18.3" && \
cd $HOME && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Binary   25.01.23
```console 
git clone https://github.com/Canto-Network/Canto
cd Canto
git checkout v5.0.0
make install

```
`cantod version --long | head`
- version: 5.0.0
- commit:  

## Initialisation
```console
cantod init <moniker-name> --chain-id canto_7700-1
```
## Add wallet
```console
cantod keys add <walletName>
cantod keys add <walletName> --recover
```
# Genesis
```console
curl -s http://164.90.154.41:26657/genesis | jq ".result.genesis" > $HOME/.cantod/config/genesis.json
```

`sha256sum $HOME/.cantod/config/genesis.json`
- 5048ba449ae348682fd86840452e88bd0812316279697c04ad288a9059f12f59  genesis.json

### Pruning (optional) one command
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.cantod/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.cantod/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.cantod/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.cantod/config/app.toml
```
### Indexer (optional) one command
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.cantod/config/config.toml

### Set up the minimum gas price and Peers/Seeds/Filter peers
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025acanto\"/;" ~/.cantod/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.cantod/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.cantod/config/config.toml

peers="ef45c32b232b772dd82b2f801f0e6abd3842e66c@164.90.154.41:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.cantod/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.cantod/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.cantod/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.cantod/config/config.toml
```

## Download addrbook
```console
wget -O $HOME/.cantod/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Canto/addrbook.json"
```

# Create a service file
```console
sudo tee /etc/systemd/system/cantod.service > /dev/null <<EOF
[Unit]
Description=cantod
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cantod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

# SnapShot 09.10.22 (0.1 GB) block height --> 1109858
```bash
# install the node as standard, but do not launch. Then we delete the .data directory and create an empty directory
sudo systemctl stop cantod
rm -rf $HOME/.cantod/data/
mkdir $HOME/.cantod/data/

# download archive
cd $HOME
wget http://canto.snapshot.stavr.tech:6001/cantodata.tar.gz

# unpack the archive
tar -C $HOME/ -zxvf cantodata.tar.gz --strip-components 1
# !! IMPORTANT POINT. If the validator was created earlier. Need to reset priv_validator_state.json  !!
wget -O $HOME/.cantod/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/Canto/priv_validator_state.json"
cd && cat .cantod/data/priv_validator_state.json
{
  "height": "0",
  "round": 0,
  "step": 0
}
# after unpacking, run the node
# don't forget to delete the archive to save space
cd $HOME
rm cantodata.tar.gz
# start the node
sudo systemctl restart cantod && journalctl -u cantod -f -o cat
```
# StateSync
```bash
SNAP_RPC="https://canto-rpc.polkachu.com:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.cantod/config/config.toml
cantod tendermint unsafe-reset-all --home $HOME/.cantod
sudo systemctl restart cantod && journalctl -u cantod -f -o cat
```

# Start node (one command)
```console
sudo systemctl daemon-reload && \
sudo systemctl enable cantod && \
sudo systemctl restart cantod && \
sudo journalctl -u cantod -f -o cat
```

## Create validator
```
cantod tx staking create-validator \
--amount=1000000000000000000acanto \
--pubkey=$(cantod tendermint show-validator) \
--moniker=<moniker> \
--chain-id=canto_7700-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees 500acanto \
--from=<walletName> \
--identity="" \
--website="" \
--details="" \
-y
```

### Delete node (one command)
```
sudo systemctl stop cantod && \
sudo systemctl disable cantod && \
rm /etc/systemd/system/cantod.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .cantod && \
rm -rf Canto && \
rm -rf $(which cantod)
```

#
### Sync Info
```bash
cantod status 2>&1 | jq .SyncInfo
```
### NodeINfo
```bash
cantod status 2>&1 | jq .NodeInfo
```
### Check node logs
```bash
sudo journalctl -u cantod -f -o cat
```
### Check Balance
```bash
cantod query bank balances canto...address1yjgn7z09ua9vms259j
```


