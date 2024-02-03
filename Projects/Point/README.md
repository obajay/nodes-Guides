[🔥OUR VALIDATOR🔥](https://restake.app/point/pointvaloper1kxrjwh009nhg8ddqrr8hwrj7nenr4zp0sj4j25)
=

# Point Network Mainnet guide
![point](https://github.com/obajay/nodes-Guides/assets/44331529/91c46183-f77c-4e6b-8a23-68232c5cb081)


[WebSite](https://pointnetwork.io/)\
[GitHub](https://github.com/pointnetwork)
=
[EXPLORER](https://explorer.stavr.tech/Point-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O pointm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Point/pointm && chmod +x pointm && ./pointm
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

# Build 17.11.23
```python
cd $HOME
git clone https://github.com/pointnetwork/point-chain
cd point-chain
git checkout tags/v0.0.5
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`pointd version --long | head`
- version: 0.0.5
- commit: b37c22b36b291403cca1205c849514a7a1b5616d

```python
pointd init STAVRguide --chain-id point_10687-1
pointd config chain-id point_10687-1
```    

## Create/recover wallet
```python
pointd keys add <walletname>
            OR
pointd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.pointd/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Point/genesis.json"

```
`sha256sum $HOME/.pointd/config/genesis.json`
+ d655a35633530c3f9d65d0c65ad287b613bba68218fd7c39549b48b958eec14f

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0apoint\"/;" ~/.pointd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.pointd/config/config.toml
peers="4896c8b474560ff359edd9e2a1e705b0513180e2@144.76.97.251:34666"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.pointd/config/config.toml
seeds="8673c1f04c29c464189e8bf29e51fb0b38da2f19@rpc-mainnet-1.point.space:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.pointd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.pointd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.pointd/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.pointd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.pointd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.pointd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.pointd/pointd/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.pointd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.pointd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Point/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/pointd.service > /dev/null <<EOF
[Unit]
Description=pointd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which pointd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Point Mainnet
```python
SNAP_RPC=https://point.rpc.m.stavr.tech:443
SEEDS=f675d544f5e6b8bc7ef9923d6f594dd0a3570190@point.peer.stavr.tech:1056
cp $HOME/.pointd/data/priv_validator_state.json $HOME/.pointd/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.pointd/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.pointd/config/config.toml
pointd tendermint unsafe-reset-all --home $HOME/.pointd --keep-addr-book
mv $HOME/.pointd/priv_validator_state.json.backup $HOME/.pointd/data/priv_validator_state.json
sudo systemctl restart pointd && journalctl -u pointd -f -o cat
```
# SnapShot Mainnet (~0.2GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop pointd
cp $HOME/.pointd/data/priv_validator_state.json $HOME/.pointd/priv_validator_state.json.backup
rm -rf $HOME/.pointd/data
curl -o - -L http://point.snapshot.stavr.tech:3/pointd/pointd-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.pointd --strip-components 2
mv $HOME/.pointd/priv_validator_state.json.backup $HOME/.pointd/data/priv_validator_state.json
wget -O $HOME/.pointd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Point/addrbook.json"
sudo systemctl restart pointd && journalctl -u pointd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable pointd
sudo systemctl restart pointd && sudo journalctl -u pointd -f -o cat
```

### Create validator
```python
pointd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000000000000000apoint \
--pubkey $(pointd tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id point_10687-1 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Point)
=

## Delete node
```python
sudo systemctl stop pointd
sudo systemctl disable pointd
rm /etc/systemd/system/pointd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf point-chain
rm -rf .pointd
rm -rf $(which pointd)
```
#
### Sync Info
```python
pointd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
pointd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u pointd -f -o cat
```
### Check Balance
```python
pointd query bank balances pointd...addressjkl1yjgn7z09ua9vms259j
```
