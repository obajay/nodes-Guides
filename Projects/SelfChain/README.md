# SelfChain Testnet guide

![Selfchain](https://github.com/obajay/nodes-Guides/assets/44331529/078afe4a-e5e8-4754-a617-4c25bec394d2)

[WebSite](https://selfchain.xyz/)\
[Docs](https://docs.selfchain.xyz/nodes-and-validators/node-setup-guide)
=
[EXPLORER 1](https://explorer.stavr.tech/Selfchain-testnet) \
[EXPLORER 2](https://explorer-devnet.selfchain.xyz/self/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O selft https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SelfChain/selft && chmod +x selft && ./selft
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y

```

## GO 1.20.3
```python
ver="1.20.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 13.09.23
```python
cd $HOME
wget https://ss-t.selfchain.nodestake.org/selfchaind
chmod +x selfchaind
mv selfchaind /root/go/bin

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`selfchaind version --long | grep -e version -e commit`
- version:
- commit:

```python
selfchaind init STAVRguide --chain-id self-dev-1
selfchaind config chain-id self-dev-1
```    

## Create/recover wallet
```python
selfchaind keys add <walletname>
  OR
selfchaind keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.selfchain/config/genesis.json "https://raw.githubusercontent.com/hotcrosscom/selfchain-genesis/main/networks/devnet/genesis.json"
```
`sha256sum $HOME/.selfchain/config/genesis.json`
+ b6defd85b263b4cf06e09319d513e660729a8aea4d71f185fee783d1117d6c98

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uself\"/;" ~/.selfchain/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.selfchain/config/config.toml
PEERS="$(curl -sS http://157.230.119.165:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | sed -z 's|\n|,|g;s|.$||')"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.selfchain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.selfchain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.selfchain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.selfchain/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.selfchain/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.selfchain/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.selfchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SelfChain/addrbook.json"
```

# Create a service file
```python
tee /etc/systemd/system/selfchaind.service > /dev/null <<EOF
[Unit]
Description=selfchaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which selfchaind) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync SelfChain Testnet
```python
SNAP_RPC=https://rpc-t.self.nodestake.top:443
peers="fda47662b03b41799e58499fac0afbaf68c02a1f@174.138.180.190:61256"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.selfchain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.selfchain/config/config.toml
selfchaind tendermint unsafe-reset-all --home $HOME/.selfchain --keep-addr-book
sudo systemctl restart selfchaind && sudo journalctl -u selfchaind -f -o cat
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOOON
```

## Start
```python
systemctl daemon-reload
systemctl enable selfchaind
systemctl restart selfchaind && journalctl -u selfchaind -f -o cat
```

### Create validator
```python
selfchaind tx staking create-validator \
  --amount=1000000uself \
  --pubkey=$(selfchaind tendermint show-validator) \
  --moniker="STAVRGuide" \
  --details="" \
  --identity="" \
  --website="" \
  --chain-id="self-dev-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.50" \
  --commission-max-change-rate="0.2" \
  --min-self-delegation="1" \
  --from=Yourwallet -y
```

## Delete node
```python
systemctl stop selfchaind
systemctl disable selfchaind
rm /etc/systemd/system/selfchaind.service
systemctl daemon-reload
cd $HOME
rm -rf .selfchain
rm -rf $(which selfchaind)
```
#
### Sync Info
```python
selfchaind status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
selfchaind status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u selfchaind -f -o cat
```
### Check Balance
```python
selfchaind query bank balances self...addressjkl1yjgn7z09ua9vms259j
```
