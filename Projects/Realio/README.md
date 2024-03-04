<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 3199881.4 | 0.862788 | 2760819.311023217284467811 |

<!-- END_TABLE -->





























































[🔥OUR VALIDATOR🔥](https://restake.app/realio/realiovaloper1n99gv9edgtvktcpxld6x9cp6zvq7e28mzjwwg4)
=

# Realio MAINNET guide
![realio](https://user-images.githubusercontent.com/44331529/194716535-b96b71ad-b191-4c28-a112-1e71e0859e0a.png)

[Website](https://www.realio.fund/)
=
[EXPLORER](https://explorer.stavr.tech/Realio-Mainnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 160GB    |


# 1) Auto_install script
```python
wget -O realio https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Realio/realio && chmod +x realio && ./realio
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
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

# Build 11.07.23
```python
cd $HOME
git clone https://github.com/realiotech/realio-network.git
cd realio-network
git checkout v0.8.3
make install
```
`realio-networkd version --long`
- version: 0.8.3
- commit: 55f63ff6ef1d98997106aab16e6accff43f40755

```python
realio-networkd init STAVRguide --chain-id realionetwork_3301-1
realio-networkd config chain-id realionetwork_3301-1
```    

## Create/recover wallet
```python
realio-networkd keys add <walletname>
   OR
realio-networkd keys add <walletname> --recover
```

## Download Genesis

```python
wget -O $HOME/.realio-network/config/genesis.json https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Realio/genesis.json
```
`sha256sum $HOME/.realio-network/config/genesis.json`
+ c255d0a493ec596b9b7c29280989b7348681e195b8b092af96631c2bf235b1c8

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ario\"/" $HOME/.realio-network/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.realio-network/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.realio-network/config/config.toml
peers="b2e50a471151aecedde282055a8f0e829aa2170b@65.109.29.224:28656,759b796d1f7c8c8362b525aaad2531591762723a@88.198.32.17:46656,5d2c9ea486a09700435ee1c0ba5291f8f1078c96@10.233.89.226:26656,4361e0e3f73ece1e6fcb9f603f0ba4ccd8ae957b@142.132.202.50:39656,9521958ef1eea934bba7f28376b7341e4dbb5f36@65.109.104.118:60856,00b261d9c9b845ce42964a3a3f6c68173875e981@65.109.28.177:30656,2c832dcd9e41d988fadf8d1af8d95640ce009398@realio.sergo.dev:12263,2e594b4782b7273ebebe47351885842c85abe8f5@65.108.229.93:32656,704eb376ec58ce6b4d1df7dfd7f0be7e79d5f200@5.9.147.22:23656,271f194229b4ee9be89777daa3ef8201553865cc@mainnet-realio.konsortech.xyz:35656,6e148794b697c64f54956ff18ca3d22fc9d95c96@148.113.6.169:30656,4a98ef79d9c80016766e247b10afe46f4cdb9892@95.216.114.212:18656,a09acd01e40c94b58cb9109fa74ce53c2220fd26@161.97.182.71:46656,cd9d9af6b7a99af3c5c920f7a054d37e297222e4@65.108.224.156:13656,daea809589ac871c6c9f450ca1cdfd5ab2320e06@57.128.110.81:26656,b09d477f5b59e5e99632ad3a8a11806381efa46f@realio.peers.stavr.tech:21096,e9cfaccc92b425fc48f2671ae9fab25c3d25926c@142.132.194.157:26557,d99c807a58f876684618af016409a09186065851@173.249.59.70:32656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.realio-network/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.realio-network/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.realio-network/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.realio-network/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.realio-network/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.realio-network/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.realio-network/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.realio-network/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.realio-network/config/config.toml
```
## Download addrbook
```python
wget -O $HOME/.realio-network/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Realio/addrbook.json"
```

# StateSync Mainnet
```python
SNAP_RPC=https://realio.rpc.m.stavr.tech:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.realio-network/config/config.toml
realio-networkd tendermint unsafe-reset-all --home /root/.realio-network
wget -O $HOME/.realio-network/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Realio/addrbook.json"
sudo systemctl restart realio-networkd && journalctl -u realio-networkd -f -o cat
```
# SnapShot (~0.1 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop realio-networkd
cp $HOME/.realio-network/data/priv_validator_state.json $HOME/.realio-network/priv_validator_state.json.backup
rm -rf $HOME/.realio-network/data
curl -o - -L http://realio.snapshot.stavr.tech:1029/realio/realio-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.realio-network --strip-components 2
mv $HOME/.realio-network/priv_validator_state.json.backup $HOME/.realio-network/data/priv_validator_state.json
wget -O $HOME/.realio-network/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Realio/addrbook.json"
sudo systemctl restart realio-networkd && journalctl -u realio-networkd -f -o cat
```


# Create a service file
```python
sudo tee /etc/systemd/system/realio-networkd.service > /dev/null <<EOF
[Unit]
Description=realio
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which realio-networkd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload && sudo systemctl enable realio-networkd
sudo systemctl restart realio-networkd && sudo journalctl -u realio-networkd -f -o cat
```

### Create validator
```python
realio-networkd tx staking create-validator \
  --amount=1000000000000000000ario \
  --pubkey=$(realio-networkd tendermint show-validator) \
  --moniker="STAVRguide" \
  --chain-id=realionetwork_3301-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.1" \
  --min-self-delegation="1" \
  --fees 5000000000000000ario \
  --gas 800000
  --from=<walletName>
```
[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Realio)
=

## Delete node
```python
sudo systemctl stop realio-networkd && \
sudo systemctl disable realio-networkd && \
rm /etc/systemd/system/realio-networkd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf realio-network && \
rm -rf .realio-network && \
rm -rf $(which realio-networkd)
```

