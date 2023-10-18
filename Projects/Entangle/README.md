# Entangle Testnet guide

![Entangle](https://github.com/obajay/nodes-Guides/assets/44331529/933f67f0-6a06-49f9-a27d-bee1b1855190)

[WebSite](https://www.entangle.fi/)\
[GitHub](https://github.com/Entangle-Protocol/entangle-blockchain)
=
[EXPLORER](https://explorer.stavr.tech/Entangle-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O entaglet https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Entangle/entaglet && chmod +x entaglet && ./entaglet
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

# Build 16.09.23
```python
cd $HOME
git clone https://github.com/Entangle-Protocol/entangle-blockchain
cd entangle-blockchain
make install
```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`version --long | grep -e commit -e version`
- version: 1.0.1
- commit: 134d9e558aee52abe81fa825272d118f293036f6

```python
entangled init STAVRguide --chain-id entangle_33133-1
entangled config chain-id entangle_33133-1
```    

## Create/recover wallet
```python
entangled keys add <walletname>
  OR
entangled keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.entangled/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Entangle/genesis.json"


```
`sha256sum $HOME/.entangled/config/genesis.json`
+ ca032fabda646ad98f4709e1e6fd4f06b31435b4a37d62a3539793944bfda692

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0001aNGL\"/;" ~/.entangled/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.entangled/config/config.toml
peers="76492a1356c14304bdd7ec946a6df0b57ba51fe2@json-rpc.testnet.entangle.fi:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.entangled/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.entangled/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 20/g' $HOME/.entangled/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 20/g' $HOME/.entangled/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.entangled/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.entangled/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.entangled/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.entangled/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.entangled/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.entangled/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Entangle/addrbook.json"
```

# Create a service file
```python
tee /etc/systemd/system/entangled.service > /dev/null <<EOF
[Unit]
Description=entangled
After=network-online.target

[Service]
User=$USER
ExecStart=$(which entangled) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Entangle Testnet
```python
SOOON
```
# SnapShot Testnet (~0.7GB) Archive SnapShot
```python
cd $HOME
apt install lz4
sudo systemctl stop entangled
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" ~/.entangled/config/config.toml
cp $HOME/.entangled/data/priv_validator_state.json $HOME/.entangled/priv_validator_state.json.backup
rm -rf $HOME/.entangled/data
curl -o - -L https://entangle.snapshot.stavr.tech/entagle/entagle-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.entangled --strip-components 2
mv $HOME/.entangled/priv_validator_state.json.backup $HOME/.entangled/data/priv_validator_state.json
wget -O $HOME/.entangled/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Entangle/addrbook.json"
sudo systemctl restart entangled && journalctl -u entangled -f -o cat
```

## Start
```python
systemctl daemon-reload
systemctl enable entangled
systemctl restart entangled && journalctl -u entangled -f -o cat
```

### Create validator
```python
entangled tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000000000000000aNGL \
--pubkey $(entangled tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_Guide" \
--chain-id entangle_33133-1 \
--gas=500000 \
--gas-prices="10aNGL" \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop entangled
sudo systemctl disable entangled
rm /etc/systemd/system/entangled.service
sudo systemctl daemon-reload
cd $HOME
rm -rf entangle-blockchain
rm -rf .entangled
rm -rf $(which entangled)
```
#
### Sync Info
```python
entangled status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
entangled status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u entangled -f -o cat
```
### Check Balance
```python
entangled query bank balances ethm...addressjkl1yjgn7z09ua9vms259j
```
