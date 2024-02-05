# Crowd Control testnet guide

![Crowd Control](https://user-images.githubusercontent.com/44331529/180597315-e25b1929-8973-4149-b2c6-b9086c1787bd.png)

[EXPLORER 1](http://explorer.stavr.tech/CARDCHAIN/staking) \
[EXPLORER 2](https://explorer.bccnodes.com/cardchain/staking)
=
- **Minimum hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   2| 4GB  | 100GB    |

# 1) Auto_install script 
```python
wget -O crowd https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crowd_Control/crowd && chmod +x crowd && ./crowd
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.21.3 (one command)
```python
ver="1.21.3"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 04.01.24
```python
git clone https://github.com/DecentralCardGame/Cardchain
wget -O Cardchaind https://github.com/DecentralCardGame/Cardchain/releases/download/v0.13.0/Cardchaind
chmod +x Cardchaind
mv $HOME/Cardchaind /usr/local/bin
```
`Cardchaind version --long | grep -e commit -e version`
+ version: 0.13.0
+ commit: c42ac40f7d96cda491bee605f6678236313219fd
    
# Init node and download Genesis
```python
Cardchaind init STAVR_guide --chain-id cardtestnet-8
Cardchaind config chain-id cardtestnet-8
wget http://45.136.28.158:3000/genesis.json -O $HOME/.Cardchain/config/genesis.json
```
## Create/recover wallet
```python
Cardchaind keys add <walletname>
        OR
Cardchaind keys add <walletname> --recover
```

## Download addrbook
```python
wget -O $HOME/.Cardchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crowd_Control/addrbook.json"
```

## Minimum gas price/Peers/Seeds
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ubpf\"/;" ~/.Cardchain/config/app.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.Cardchain/config/config.toml
peers="58bb9f1dcde0408fe4c3e7f8c6ccb7f8e410ca9c@202.61.225.157:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.Cardchain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.Cardchain/config/config.toml
```


### Pruning (optional)
```python
pruning="custom" &&
pruning_keep_recent="1000" &&
pruning_keep_every="0" &&
pruning_interval="10" &&
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain/config/app.toml &&
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain/config/app.toml &&
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain/config/app.toml &&
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain/config/app.toml
```
### Indexer (optional)
```python
ndexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.Cardchain/config/config.toml
```

# Service file
```python
sudo tee <<EOF >/dev/null /etc/systemd/system/Cardchaind.service
[Unit]
Description=Cardchain Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which Cardchaind) start
Restart=always
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
## StateSync 
```python
SNAP_RPC=https://crowd.rpc.t.stavr.tech:443
PEERS="ec585d7fb38b67619dcb79aad90722f0eaf0faa3@crowd.peer.stavr.tech:21206"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.Cardchain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height) \
&& BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)) \
&& TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.Cardchain/config/config.toml; \
Cardchaind tendermint unsafe-reset-all
wget -O $HOME/.Cardchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crowd_Control/addrbook.json"
sudo systemctl restart Cardchaind && journalctl -u Cardchaind -f -o cat
```

## SnapShot (~0.3 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" ~/.Cardchain/config/config.toml
sudo systemctl stop Cardchaind
cp $HOME/.Cardchain/data/priv_validator_state.json $HOME/.Cardchain/priv_validator_state.json.backup
rm -rf $HOME/.Cardchain/data
curl -o - -L http://crowd.snapshot.stavr.tech:1013/crowd/crowd-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.Cardchain --strip-components 2
mv $HOME/.Cardchain/priv_validator_state.json.backup $HOME/.Cardchain/data/priv_validator_state.json
wget -O $HOME/.Cardchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Crowd_Control/addrbook.json"
sudo systemctl restart Cardchaind && journalctl -u Cardchaind -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable Cardchaind
sudo systemctl restart Cardchaind && sudo journalctl -u Cardchaind -f -o cat
```

## Create validator
```python
Cardchaind tx staking create-validator \
--amount 1000000ubpf \
--from <walletName> \
--commission-max-change-rate "0.2" \
--commission-max-rate "1" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--details="" \
--identity="" \
--pubkey  $(Cardchaind tendermint show-validator) \
--moniker STAVR_Guide \
--fees 300ubpf \
--chain-id cardtestnet-8 -y
```
[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Crowd_Control)
=


## Delete node
```python
sudo systemctl stop Cardchaind
sudo rm /etc/systemd/system/Cardchaind.service
sudo rm -rf $HOME/.Cardchain/
sudo rm -rf Testnet
sudo rm -rf $(which Cardchaind)
```

# Cardchain Parameters:

| Parameter:                    | Value:      | 
|-------------------------------|-------------|
| Voting Rights Expiration Time |  100000     |
| Set Size                      |  5          |
| Set Price	                    |  10000000   |
| Active Sets Amount	        |  3          |
| Set Creation Fee	            |  5000000000 |
| Collateral Deposit	        |  50000000   |
| Winner Reward	                |  1000000    |
| Hourly Faucet	                |  50000000   |
| Inflation Rate	            |  1.1        |
| Rares Per Pack	            |  1          |
| Commons Per Pack	            |  4          |
| Un Commons Per Pack	        |  2          |
| Trial Period	                |  168000     |
| Game Vote Ratio	            |  20         |
| Card Auction Price Reduction Period|  20    |
| Air Drop Value	            |  5000000    |
| Air Drop Max Block Height	    |  5000000    |
| Trial Vote Reward	            |  1000000    |
| Vote Pool Fraction	        |  1000000    |
| Voting Reward Cap	            |  1000000    |
| Match Worker Delay	        |  1200       |
| Rare Drop Ratio	            |  150        |
| Exceptional Drop Ratio	    |  50         |
| Unique Drop Ratio	            |  1          |

<h2 align="center"> We scan nodes in real time every 4 hours. And we provide the final result of RPC endpoints.
We cannot influence the operation of these nodes in any way. </h2>

```python
If Voting Power is higher than 0 --> then the Node is a validator of the network and may be subject to attack and be a potential threat to the chain. 
We marked such validators with a red symbol
```
[Testnet raw json](https://rpc-check.crowd.stavr.tech/crowd/rpc_result.json)
=

<table><tr><th>IP-Address</th><th>Network</th><th>Moniker</th><th>Latest Block Height</th><th>Earliest Block Height</th><th>Catching Up</th><th>Tx Index</th><th>Voting Power</th><th>Scan Time</th></tr><tr><td>65.21.53.39:16657</td><td>cardtestnet-8</td><td>Yurbason 🔴</td><td>486787</td><td>1</td><td>False</td><td>off</td><td>490</td><td>2024-02-05T08:55:28.169147231CET</td></tr><tr><td>5.75.153.46:21207</td><td>cardtestnet-8</td><td>Aurie 🔴</td><td>486787</td><td>1</td><td>False</td><td>on</td><td>105</td><td>2024-02-05T08:55:28.563593929CET</td></tr><tr><td>144.76.97.251:38657</td><td>cardtestnet-8</td><td>AlxVoy 🔴</td><td>486787</td><td>1</td><td>False</td><td>on</td><td>500</td><td>2024-02-05T08:55:29.560950636CET</td></tr><tr><td>45.136.28.158:26657</td><td>cardtestnet-8</td><td>yesyoulikeCC 🟢</td><td>486787</td><td>1</td><td>False</td><td>on</td><td>0</td><td>2024-02-05T08:55:29.933445320CET</td></tr><tr><td>86.48.0.190:13657</td><td>cardtestnet-8</td><td>AnatoliaService 🟢</td><td>486787</td><td>87101</td><td>False</td><td>off</td><td>0</td><td>2024-02-05T08:55:30.973819226CET</td></tr><tr><td>109.123.242.217:21207</td><td>cardtestnet-8</td><td>STAVR 🔴</td><td>486787</td><td>347001</td><td>False</td><td>on</td><td>635</td><td>2024-02-05T08:55:30.593291624CET</td></tr><tr><td>167.86.68.204:39657</td><td>cardtestnet-8</td><td>cryptobtcbuyer 🔴</td><td>486787</td><td>386787</td><td>False</td><td>off</td><td>499</td><td>2024-02-05T08:55:29.054541215CET</td></tr><tr><td>167.235.155.8:21207</td><td>cardtestnet-8</td><td>STAVR-Service 🟢</td><td>486787</td><td>484001</td><td>False</td><td>on</td><td>0</td><td>2024-02-05T08:55:30.291997755CET</td></tr></table>
