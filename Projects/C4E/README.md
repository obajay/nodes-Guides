[🔥OUR VALIDATOR🔥](https://restake.app/chain4energy/c4evaloper1gc6vs2j3g2jpy2mzrwpmmwju9eafhn0gwx8lv8)
=

<h1 align="center"> 🔥C4E MAINNET guide🔥</h1>

![c4e (2)](https://user-images.githubusercontent.com/44331529/216780015-d723d176-ae86-403e-aab6-7d7e6254a144.png)


[WebSite](https://c4e.io/) \
[GitHub](https://github.com/chain4energy)
=
[EXPLORER 1](https://explorer.stavr.tech/C4E/staking) \
[EXPLORER 2](https://exp.utsa.tech/c4e/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB    |


# 1) Auto_install script
```python
wget -O c4 https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/C4E/c4 && chmod +x c4 && ./c4
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

# Build 28.02.24
```python
cd $HOME
git clone https://github.com/chain4energy/c4e-chain
cd c4e-chain
git checkout v1.3.1
make install
```
*******🟢UPDATE🟢******* 28.02.24
```python
cd $HOME
wget https://github.com/chain4energy/c4e-chain/releases/download/v1.3.1/c4ed_v1.3.1_linux_amd64.tar.gz
tar -xvf c4ed_v1.3.1_linux_amd64.tar.gz
rm -rf c4ed_v1.3.1_linux_amd64.tar.gz
chmod +x c4ed
mv $HOME/c4ed $(which c4ed)
c4ed version --long
#commit: 55fe5c3c1a1b9a8489d39e765413315c5b61c4f7
#version: 1.3.1
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat

```

`c4ed version --long`
- version: 1.3.1
- commit: 55fe5c3c1a1b9a8489d39e765413315c5b61c4f7

```python
c4ed init STAVRguide --chain-id perun-1
c4ed config chain-id perun-1
```    

## Create/recover wallet
```python
c4ed keys add <walletname>
  OR
c4ed keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/C4E/genesis.json -O $HOME/.c4e-chain/config/genesis.json
```

`sha256sum $HOME/.c4e-chain/config/genesis.json`
+ 6c736993a681a6759d3ec41550995fe04f48dd332d03375d879f3b464c6ceabf

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uc4e\"/" $HOME/.c4e-chain/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.c4e-chain/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.c4e-chain/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.c4e-chain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
```

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.c4e-chain/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.c4e-chain/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.c4e-chain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/C4E/addrbook.json"
```
## StateSync
```python
SNAP_RPC=https://c4e.rpc.m.stavr.tech:443
peers="5ed0b8f7989d34438f71ccc74b0ab0fbf763a475@c4e.peer.stavr.tech:17096"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.c4e-chain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.c4e-chain/config/config.toml
c4ed tendermint unsafe-reset-all --home /root/.c4e-chain --keep-addr-book
sudo systemctl restart c4ed && journalctl -u c4ed -f -o cat
```
## SnapShot (~0.1 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop c4ed
cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
rm -rf $HOME/.c4e-chain/data
curl -o - -L http://c4e.snapshot.stavr.tech:1018/c4e/c4e-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.c4e-chain --strip-components 2
mv $HOME/.c4e-chain/priv_validator_state.json.backup $HOME/.c4e-chain/data/priv_validator_state.json
wget -O $HOME/.c4e-chain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/C4E/addrbook.json"
sudo systemctl restart c4ed && journalctl -u c4ed -f -o cat
```

# Create a service file
```python
sudo tee /etc/systemd/system/c4ed.service > /dev/null <<EOF
[Unit]
Description=c4e
After=network-online.target

[Service]
User=$USER
ExecStart=$(which c4ed) start
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
sudo systemctl enable c4ed
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```

### Create validator
```python
c4ed tx staking create-validator \
  --amount 1000000uc4e \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(c4ed tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id perun-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/C4E)
=

## Delete node
```python
sudo systemctl stop c4ed && \
sudo systemctl disable c4ed && \
rm /etc/systemd/system/c4ed.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf c4e-chain && \
rm -rf .c4e-chain && \
rm -rf $(which c4ed)
```
#
### Sync Info
```python
source $HOME/.bash_profile
c4ed status 2>&1 | jq .SyncInfo
```
### Node Info
```python
c4ed status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u c4ed -f -o cat
```
### Check Balance
```python
c4ed query bank balances c4e...address1yjgn7z09ua9vms259j
```

<h1 align="center"> 📚Useful commands📚 </h1>

# ⚙️Service

#### Info
```python
c4ed status 2>&1 | jq .NodeInfo
c4ed status 2>&1 | jq .SyncInfo
c4ed status 2>&1 | jq .ValidatorInfo
```
#### Check node logs
```python
sudo journalctl -fu c4ed -o cat
```
#### Check service status
```python
sudo systemctl status c4ed
```
#### Restart service
```python
sudo systemctl restart c4ed
```
#### Stop service
```python
sudo systemctl stop c4ed
```
#### Start service
```python
sudo systemctl start c4ed
```
#### reload/disable/enable
```python
sudo systemctl daemon-reload
sudo systemctl disable c4ed
sudo systemctl enable c4ed
```
#### Your Peer
```python
echo $(c4ed tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.c4e-chain/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

# 🥅Working with keys

#### New Key or Recover Key
```python
c4ed keys add Wallet_Name
      OR
c4ed keys add Wallet_Name --recover
```
#### Check all keys
```python
c4ed keys list
```
#### Check Balance
```python
c4ed query bank balances c4e...addressjkl1yjgn7z09ua9vms259j
```
#### Delete Key
```python
c4ed keys delete Wallet_Name
```
#### Export Key
```python
c4ed keys export wallet
```
#### Import Key
```python
c4ed keys import wallet wallet.backup
```

# 🚀Validator Management

#### Edit Validator
```python
c4ed tx staking edit-validator \
--new-moniker "Your_Moniker" \
--identity "Keybase_ID" \
--details "Your_Description" \
--website "Your_Website" \
--security-contact "Your_Email" \
--chain-id perun-1 \
--commission-rate 0.05 \
--from Wallet_Name \
--gas 350000 -y
```

#### Your Valoper-Address
```python
c4ed keys show Wallet_Name --bech val
```
#### Your Valcons-Address
```python
c4ed tendermint show-address
```
#### Your Validator-Info
```python
c4ed query staking validator c4evaloperaddress......
```
#### Jail Info
```python
c4ed query slashing signing-info $(c4ed tendermint show-validator)
```
#### Unjail
```python
c4ed tx slashing unjail --from Wallet_name --chain-id perun-1 --gas 350000 -y
```
#### Active Validators List
```python
c4ed q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Inactive Validators List
```python
c4ed q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Check that your key matches the validator  (Win - Good.  Lose - Bad)
```python
VALOPER=Enter_Your_valoper_Here
[[ $(c4ed  q staking validator $VALOPER -oj | jq -r .consensus_pubkey.key) = $(c4ed status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\nYou win\n" || echo -e "\nYou lose\n"
```

#### Withdraw all rewards from all validators
```python
c4ed tx distribution withdraw-all-rewards --from Wallet_Name --chain-id perun-1 --gas 350000 -y
```
#### Withdraw and commission from your Validator
```python
c4ed tx distribution withdraw-rewards c4evaloper1amxp0k0hg4edrxg85v07t9ka2tfuhamhldgf8e --from Wallet_Name --gas 350000 --chain-id=perun-1 --commission -y
```
#### Delegate tokens to your validator
```python
c4ed tx staking delegate Your_c4evalpoer........ "100000000"uc4e --from Wallet_Name --gas 350000 --chain-id=perun-1 -y
```
#### Delegate tokens to different validator
```python
c4ed tx staking delegate c4evalpoer........ "100000000"uc4e --from Wallet_Name --gas 350000 --chain-id=perun-1 -y
```
#### Redelegate tokens to another validator
```python
c4ed tx staking redelegate Your_c4evalpoer........ c4evalpoer........ "100000000"uc4e --from Wallet_Name --gas 350000  --chain-id=perun-1 -y
```

#### Unbond tokens from your validator or different validator
```python
c4ed tx staking unbond Your_c4evalpoer........ "100000000"uc4e --from Wallet_Name --gas 350000 --chain-id=perun-1 -y
c4ed tx staking unbond c4evalpoer........ "100000000"uc4e --from Wallet_Name --gas 350000 --chain-id=perun-1 -y
```

#### Transfer tokens from wallet to wallet
```python
c4ed tx bank send Your_c4eaddress............ c4eaddress........... "1000000000000000000"uc4e --gas 350000 --chain-id=perun-1 -y
```

# 📝Governance

#### View all proposals
```python
c4ed query gov proposals
```

#### View specific proposal
```python
c4ed query gov proposal 1
```

#### Vote yes
```python
c4ed tx gov vote 1 yes --from Wallet_Name --gas 350000  --chain-id=perun-1 -y
```
#### Vote no
```python
c4ed tx gov vote 1 no --from Wallet_Name --gas 350000  --chain-id=perun-1 -y
```
#### Vote abstain
```python
c4ed tx gov vote 1 abstain --from Wallet_Name --gas 350000  --chain-id=perun-1 -y
```
#### Vote no_with_veto
```python
c4ed tx gov vote 1 no_with_veto --from Wallet_Name --gas 350000  --chain-id=perun-1 -y
```
