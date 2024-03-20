# Sge Testnet guide

![sge](https://user-images.githubusercontent.com/44331529/203624604-ec312821-11cd-404e-8647-bbbcfddcaf8a.png)

[WebSite](https://sgenetwork.io/)
=
[EXPLORER](https://explorer.stavr.tech/Sge-Testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O sgge https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/Testnet/sgge && chmod +x sgge && ./sgge
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19.4

```python
GO 19.4
ver="1.19.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 21.03.24
```python
cd $HOME
git clone https://github.com/sge-network/sge
cd sge
git checkout v1.5.3
make install
```
`sged version --long`
- version: v1.5.3
- commit: 977c6dad66304040a11b7faa84782a1dacc1dc1f

```python
sged init STAVR_guide --chain-id sge-network-4
sged config chain-id sge-network-4
```    

## Create/recover wallet
```python
sged keys add <walletname>
sged keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.sge/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/Testnet/genesis.json"

```
`sha256sum $HOME/.sge/config/genesis.json`
+ 08f37f99e33fe2521ff1d1128b437f6351018bad871aedb942c9167d2d83964b

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usge\"/" $HOME/.sge/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sge/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sge/config/config.toml
peers="145d0f311ef1485f5b95eebecbc758fce01b4bb6@38.146.3.184:17756,6caabc35628a51bbf9c80ead303f13b3dfae8674@50.19.180.153:26656,51e4e7b04d2f669f5efa53e8d95891fa04e4c5b9@206.125.33.62:26656,2b4efc999c6aaad3cb2456fa5385f16f90e2c3d2@95.217.106.215:11156,31bda14eacbc1c1c537c4b7c2e8d338a06c8c5fd@57.128.37.47:26656,ef9ac611d9ca1c3a9fae22199f449d7c1082a0d9@65.108.233.109:17756"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sge/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sge/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 10/g' $HOME/.sge/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 40/g' $HOME/.sge/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" &&
pruning_keep_recent="1000" &&
pruning_keep_every="0" &&
pruning_interval="10" &&
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.sge/config/app.toml &&
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.sge/config/app.toml &&
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.sge/config/app.toml &&
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.sge/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" &&
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sge/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.sge/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/Testnet/addrbook.json"
```
## StateSync Testnet
```python
SNAP_RPC=https://sge.rpc.t.stavr.tech:443
peers="e2c5f2a902b7e6b8c006008e962ab4ddd70cdd78@sge.peers-t.stavr.tech:1146"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.sge/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.sge/config/config.toml
sged tendermint unsafe-reset-all --home /root/.sge
wget -O $HOME/.sge/config/addrbook.json "wget -O $HOME/.sge/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/Testnet/genesis.json"
systemctl restart sged && journalctl -u sged -f -o cat
```
## SnapShot (~0.2 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop sged
cp $HOME/.sge/data/priv_validator_state.json $HOME/.sge/priv_validator_state.json.backup
rm -rf $HOME/.sge/data
curl -o - -L http://sge-t.snapshot.stavr.tech:1017/sget/sget-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.sge --strip-components 2
mv $HOME/.sge/priv_validator_state.json.backup $HOME/.sge/data/priv_validator_state.json
wget -O $HOME/.sge/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/SGE/addrbook.json"
sudo systemctl restart sged && journalctl -u sged -f -o cat

```

# Create a service file
```python
sudo tee /etc/systemd/system/sged.service > /dev/null <<EOF
[Unit]
Description=sge
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sged) start
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
sudo systemctl enable sged
sudo systemctl restart sged && sudo journalctl -u sged -f -o cat
```

### Create validator
```python
sged tx staking create-validator \
  --amount 1000000usge \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(sged tendermint show-validator) \
  --moniker STAVR_guide \
  --chain-id sge-network-4 \
  --identity="" \
  --details="" \
  --website="" -y
```
[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Sge)
=

## Delete node
```bash
sudo systemctl stop sged
sudo systemctl disable sged
rm /etc/systemd/system/sged.service
sudo systemctl daemon-reload
cd $HOME
rm -rf sge
rm -rf .sge
rm -rf $(which sged)
```
#
### Sync Info
```python
sged status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
sged status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u sged -f -o cat
```
### Check Balance
```python
sged query bank balances sge...address1yjgn7z09ua9vms259j
```
