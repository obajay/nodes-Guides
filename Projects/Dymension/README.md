<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 690223.7 | 6.42 | 4431236.544534169066573756 |

<!-- END_TABLE -->









































[🔥OUR VALIDATOR🔥](https://restake.app/dymension/dymvaloper1amxp0k0hg4edrxg85v07t9ka2tfuhamhldgf8e)
=

# Dymension Mainnet guide

![dymension](https://user-images.githubusercontent.com/44331529/216242184-e602001a-8794-495a-81fc-b0d10589963e.png)


[WebSite](https://dymension.xyz/) \
[GitHub](https://github.com/dymensionxyz/testnets)
=
[EXPLORER](https://explorer.stavr.tech/Dymension-Mainnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |


# 1) Auto_install script
```python
wget -O dymm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Dymension/dymm && chmod +x dymm && ./dymm
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.20.5
```python
ver="1.20.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 06.02.24
```python
cd $HOME
git clone https://github.com/dymensionxyz/dymension.git
cd dymension
git checkout v3.0.0
make install

```
*******🟢UPDATE🟢******* 00.00.24
```python
SOOOON
```

`dymd version --long | grep -e commit -e version`
- version: v3.0.0
- commit: c3294dc8d2dce1aa8efbc967b1dfd3b0e965b095

```python
dymd init STAVR_guide --chain-id=dymension_1100-1
dymd config chain-id dymension_1100-1
```    

## Create/recover wallet
```python
dymd keys add <walletname>
dymd keys add <walletname> --recover
```

## Download Genesis
```python
wget https://github.com/dymensionxyz/networks/raw/main/mainnet/dymension/genesis.json -O $HOME/.dymension/config/genesis.json
```
`sha256sum $HOME/.dymension/config/genesis.json`
+ c25f362084db5c1480aaee93bfcb97c5328cabeda94f11ddcc74a8e183838491

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"20000000000adym\"/;" ~/.dymension/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.dymension/config/config.toml
peers="09b1a88148c16a3cc629b7cfc12fb369d7a3399a@65.108.233.90:26656,39c335604e9e9323eb177ef8c33f8ab4a4317498@85.215.125.37:26656,fb7a8f69270a7de8a3c1b1e79e194a407d305c63@84.203.117.234:26691"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.dymension/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.dymension/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.dymension/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.dymension/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.dymension/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" &&
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.dymension/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Dymension/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/dymd.service > /dev/null <<EOF
[Unit]
Description=dymd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which dymd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Dymension Mainnet
```python
SNAP_RPC=https://dym.rpc.m.stavr.tech:443
peers="e0d84deab2d0fd85f447c5c417fecbbdba584be0@dymension-m.peer.stavr.tech:17086"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.dymension/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.dymension/config/config.toml
dymd tendermint unsafe-reset-all --home /root/.dymension
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Dymension/addrbook.json"
systemctl restart dymd && journalctl -u dymd -f -o cat

```
# SnapShot Mainnet updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop dymd
cp $HOME/.dymension/data/priv_validator_state.json $HOME/.dymension/priv_validator_state.json.backup
rm -rf $HOME/.dymension/data
curl -o - -L https://dymension-m.snapshot.stavr.tech/dymension-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.dymension --strip-components 2
mv $HOME/.dymension/priv_validator_state.json.backup $HOME/.dymension/data/priv_validator_state.json
wget -O $HOME/.dymension/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Dymension/addrbook.json"
sudo systemctl restart dymd && journalctl -u dymd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable dymd
sudo systemctl restart dymd && sudo journalctl -u dymd -f -o cat
```

### Create validator
```python
dymd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.2 \
--min-self-delegation "1000000" \
--amount 1000000000000000000adym \
--pubkey $(dymd tendermint show-validator) \
--from <wallet> \
--gas 350000 \
--fees 7000000000000000adym \
--moniker="STAVR_guide" \
--chain-id="dymension_1100-1" \
--identity="" \
--website="" \
--details="" \
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Dymension)
=

## Delete node
```python
sudo systemctl stop dymd
sudo systemctl disable dymd
rm /etc/systemd/system/dymd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf dymension
rm -rf .dymension
rm -rf $(which dymd)
```

<h1 align="center"> 📚Useful commands📚 </h1>

# ⚙️Service

#### Info
```python
dymd status 2>&1 | jq .NodeInfo
dymd status 2>&1 | jq .SyncInfo
dymd status 2>&1 | jq .ValidatorInfo
```
#### Check node logs
```python
sudo journalctl -fu dymd -o cat
```
#### Check service status
```python
sudo systemctl status dymd
```
#### Restart service
```python
sudo systemctl restart dymd
```
#### Stop service
```python
sudo systemctl stop dymd
```
#### Start service
```python
sudo systemctl start dymd
```
#### reload/disable/enable
```python
sudo systemctl daemon-reload
sudo systemctl disable dymd
sudo systemctl enable dymd
```
#### Your Peer
```python
echo $(dymd tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.dymension/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

# 🥅Working with keys

#### New Key or Recover Key
```python
dymd keys add Wallet_Name
      OR
dymd keys add Wallet_Name --recover
```
#### Check all keys
```python
dymd keys list
```
#### Check Balance
```python
dymd query bank balances dym...addressjkl1yjgn7z09ua9vms259j
```
#### Delete Key
```python
dymd keys delete Wallet_Name
```
#### Export Key
```python
dymd keys export wallet
```
#### Import Key
```python
dymd keys import wallet wallet.backup
```

# 🚀Validator Management

#### Edit Validator
```python
dymd tx staking edit-validator \
--new-moniker "Your_Moniker" \
--identity "Keybase_ID" \
--details "Your_Description" \
--website "Your_Website" \
--security-contact "Your_Email" \
--chain-id dymension_1100-1 \
--commission-rate 0.05 \
--from Wallet_Name \
--gas 350000 \
--fees 7000000000000000adym -y
```

#### Your Valoper-Address
```python
dymd keys show Wallet_Name --bech val
```
#### Your Valcons-Address
```python
dymd tendermint show-address
```
#### Your Validator-Info
```python
dymd query staking validator dymdvaloperaddress......
```
#### Jail Info
```python
dymd query slashing signing-info $(dymd tendermint show-validator)
```
#### Unjail
```python
dymd tx slashing unjail --from Wallet_name --chain-id dymension_1100-1 --gas 350000 --fees 7000000000000000adym -y
```
#### Active Validators List
```python
dymd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Inactive Validators List
```python
dymd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Check that your key matches the validator  (Win - Good.  Lose - Bad)
```python
VALOPER=Enter_Your_valoper_Here
[[ $(dymd  q staking validator $VALOPER -oj | jq -r .consensus_pubkey.key) = $(dymd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\nYou win\n" || echo -e "\nYou lose\n"
```

#### Withdraw all rewards from all validators
```python
dymd tx distribution withdraw-all-rewards --from Wallet_Name --chain-id dymension_1100-1 --gas 350000 --fees 7000000000000000adym -y
```
#### Withdraw and commission from your Validator
```python
dymd tx distribution withdraw-rewards dymvaloper1amxp0k0hg4edrxg85v07t9ka2tfuhamhldgf8e --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 --commission -y
```
#### Delegate tokens to your validator
```python
dymd tx staking delegate Your_dymvalpoer........ "100000000"adym --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```
#### Delegate tokens to different validator
```python
dymd tx staking delegate dymvalpoer........ "100000000"adym --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```
#### Redelegate tokens to another validator
```python
dymd tx staking redelegate Your_dymvalpoer........ dymvalpoer........ "100000000"adym --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```

#### Unbond tokens from your validator or different validator
```python
dymd tx staking unbond Your_dymvalpoer........ "100000000"adym --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
dymd tx staking unbond dymvalpoer........ "100000000"adym --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```

#### Transfer tokens from wallet to wallet
```python
dymd tx bank send Your_dymaddress............ dymaddress........... "1000000000000000000"adym --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```

# 📝Governance

#### View all proposals
```python
dymd query gov proposals
```

#### View specific proposal
```python
dymd query gov proposal 1
```

#### Vote yes
```python
dymd tx gov vote 1 yes --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```
#### Vote no
```python
dymd tx gov vote 1 no --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```
#### Vote abstain
```python
dymd tx gov vote 1 abstain --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```
#### Vote no_with_veto
```python
dymd tx gov vote 1 no_with_veto --from Wallet_Name --gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```


# 📡IBC  transfer 
- for exapmle - DYM -> Osmosis
```python
dymd tx ibc-transfer transfer transfer channel-2 Your_OSMOaddress............ "100000"adym --from Your_DYM_Wallet_Name ---gas 350000 --fees 7000000000000000adym --chain-id=dymension_1100-1 -y
```
