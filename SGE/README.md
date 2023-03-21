# Sge Testnet guide

![sge](https://user-images.githubusercontent.com/44331529/203624604-ec312821-11cd-404e-8647-bbbcfddcaf8a.png)

[WebSite](https://sgenetwork.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/sge-testnet/staking) \
[EXPLORER 2](https://exp.nodeist.net/t-sge/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O sgge https://raw.githubusercontent.com/obajay/nodes-Guides/main/SGE/sgge && chmod +x sgge && ./sgge
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.18

```python
ver="1.18" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 16.03.23
```python
git clone https://github.com/sge-network/sge
git clone https://github.com/sge-network/networks
cd sge
git fetch --tags
git checkout v0.0.5
cd sge
go mod tidy
make install
```
`sged version --long`
- version: v0.0.5
- commit: 462ff3ad9721a1fcfd6edc63654b4b13569a6f9a

```python
sged init STAVRguide --chain-id sge-network-2
sged config chain-id sge-network-2
```    

## Create/recover wallet
```python
sged keys add <walletname>
sged keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.sge/config/genesis.json "https://raw.githubusercontent.com/sge-network/networks/master/sge-network-2/genesis.json"

```
`sha256sum $HOME/.sge/config/genesis.json`
+ d5e51eeb2e4eab83bb7272525ff7d1f605561c6b5cab0465807866149e3a1fc4

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usge\"/" $HOME/.sge/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sge/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sge/config/config.toml
peers="62b76a24869829fb3be53c25891ba37eca5994bd@95.217.224.252:26656,b29612454715a6dc0d1f0c42b426bf30f1d27738@78.46.99.50:24656,14823c9230ac2eb50fd48b7313e8ddd4c13207c6@94.130.219.37:26000,cfa86646e5eb05e111e7dde27750ff8ebe67d165@89.117.56.126:23956,43b05a6bab7ca735397e9fae2cb0ad99977cf482@34.83.191.67:26656,ddcd5fda167e6b45208faed8fd7e2f0640b4185c@52.44.14.245:26656,a05353fe9ae39dd0edbfa6341634dec781d84a5c@65.108.105.48:17756,1168931936c638e92ea6d93e2271b3fe5faee6d1@135.125.247.228:26656,27f0b281ea7f4c3db01fdb9f4cf7cc910ad240a6@209.34.205.57:26656,b4f800aa8ff11d0d7ab3f5ce19230f049dfebe4b@38.242.199.160:26656,8c74885d4310f606986c88e9613f5e48c9e154dd@65.108.2.41:56656,a13512dbb3def06f91aef81afb397db63d78b25c@51.195.89.114:20656,bbf84e77c0defea82d389e1bd0940d7718f0ee34@103.230.84.4:26656,3e644c24129e14d457e82bab3b5a16c510b12927@50.19.180.153:26656,d200a21e2b3edab24679d4544fea48471515098f@65.108.225.158:17756,dc831d440c18c4a4f72250806cd03e5b240f8935@3.15.209.96:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sge/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sge/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.sge/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.sge/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.sge/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.sge/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.sge/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.sge/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sge/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.sge/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/SGE/addrbook.json"
```
## StateSync
```python
SNAP_RPC=sge.rpc.t.stavr.tech:1157
peers="fc616f3e9dc79997e60bd915a9233b6cc81bcd0f@sge.peers.stavr.tech:1156"
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
sged tendermint unsafe-reset-all --home /root/.sge --keep-addr-book
systemctl restart sged && journalctl -u sged -f -o cat
```
## SnapShot (~0.2 GB) updated every 5 hours
```python
cd $HOME
snap install lz4
sudo systemctl stop sged
cp $HOME/.sge/data/priv_validator_state.json $HOME/.sge/priv_validator_state.json.backup
rm -rf $HOME/.sge/data
curl -o - -L http://sge.snapshot.stavr.tech:1003/sge/sge-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.sge --strip-components 2
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
  --moniker STAVRguide \
  --chain-id sge-network-2 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop sged && \
sudo systemctl disable sged && \
rm /etc/systemd/system/sged.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf sge && \
rm -rf .sge && \
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
