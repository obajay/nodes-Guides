<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 13500035.404 | 0.181745 | 2453563.934579766 |

<!-- END_TABLE -->





















[🔥OUR VALIDATOR🔥](https://restake.app/sge/sgevaloper105xysdmvy6xyz3ytyk62fgkwyczx2n4vzkyp6g)
=

# Sge Mainnet guide
![sge](https://user-images.githubusercontent.com/44331529/203624604-ec312821-11cd-404e-8647-bbbcfddcaf8a.png)

[WebSite](https://sgenetwork.io/)
=
[EXPLORER](https://explorer.stavr.tech/Sge-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O sggem https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/sggem && chmod +x sggem && ./sggem
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19

```python
ver="1.19" &&
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" &&
sudo rm -rf /usr/local/go &&
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" &&
rm "go$ver.linux-amd64.tar.gz" &&
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile &&
source $HOME/.bash_profile &&
go version
```

# Build 14.01.24
```python
cd $HOME
git clone https://github.com/sge-network/sge
cd sge
git checkout v1.3.1
make install
```
`sged version --long`
- version: v1.3.1
- commit: 4a96c146cbd06454806e04ad826601597f15ccb3 

```python
sged init STAVR_guide --chain-id sgenet-1
sged config chain-id sgenet-1
```    

## Create/recover wallet
```python
sged keys add <walletname>
sged keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.sge/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/genesis.json"
```
`sha256sum $HOME/.sge/config/genesis.json`
+ 3beb0662ade1ad80d41d992bb196770d53a939863c1fed12fa01411dfb981e0b

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usge\"/" $HOME/.sge/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sge/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sge/config/config.toml
peers="05628e99f42eb2fbacfd1f0402f96f46b88dfe6b@146.59.52.137:17756,fe527359b6b6c5ad9cc6e2f6ed3af46018b29e15@136.243.36.60:17756,7258d8c7880167fca502592b8d64110d60e99a6b@65.108.232.180:17756,752bc8c7508affd7e2af494a6bf44bcb66cf84ea@65.108.39.140:17756,88f341a9670494c3d529934dc578eec1b00f4aa1@141.94.168.85:26656,59c71e1ae0267da913d8460c10bbbb86f8003d12@85.10.201.125:36656,6c1cbeb621f04886029c7b222041f7fdb307c579@94.130.14.54:17756,0aa028990c5a135e89447e88daf65a8a590257f4@136.243.67.44:17756,4078d8f702a2ee25c8da93938940748276652696@94.130.13.186:17756,304535618b71c2fe217fe771c745443ea3d7815e@65.108.0.94:17756,ad1dce877d93f9de0d3a5c0b0f28d114242c1d3b@64.185.227.122:17756,8f4ca666d56fc883328b1aa0796342c1c1602099@64.185.226.202:17756,401a4986e78fe74dd7ead9363463ba4c704d8759@38.146.3.183:17756,a6a3ef121282dbf6d9ac70a83cba02780a6b4a5c@67.209.54.93:17756,ba0a167567c7e08f4bb1e25ec24e42a85b07a0c5@148.113.20.208:17756,8fb88c54a8175908bab4dc4122652e7480988d97@3.37.63.177:26656,cca02db11dd1c59c91d355a72c702ccb26a9f99f@3.39.189.46:26656,3fc703341935b9356addfe7b3aad8991d9c8a923@148.113.20.207:17756,e55fe14a534f8cb9a8b8fb1b4a626d867bf642bb@162.19.69.49:52656,bf01fb9d4eab9e007a47c0c3d3b423c5fb426207@65.109.108.47:17756,11a44cfe807274df4ddbce7ee61c111bcecba6f0@65.109.82.87:17756"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sge/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sge/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 10/g' $HOME/.sge/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 40/g' $HOME/.sge/config/config.toml

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
wget -O $HOME/.sge/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/addrbook.json"
```
## StateSync Mainnet
```python
SNAP_RPC=https://sge.rpc.m.stavr.tech:443
peers="58a458a7136da7e8cc55357999aa89f5fd262588@sge.peers.stavr.tech:1156"
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
wget -O $HOME/.sge/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/addrbook.json"
systemctl restart sged && journalctl -u sged -f -o cat
```
## SnapShot (~0.2 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop sged
cp $HOME/.sge/data/priv_validator_state.json $HOME/.sge/priv_validator_state.json.backup
rm -rf $HOME/.sge/data
curl -o - -L http://sge.snapshot.stavr.tech:1003/sge/sge-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.sge --strip-components 2
mv $HOME/.sge/priv_validator_state.json.backup $HOME/.sge/data/priv_validator_state.json
wget -O $HOME/.sge/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/SGE/addrbook.json"
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
  --chain-id sgenet-1 \
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
