# Warden Testnet guide

![warden](https://github.com/obajay/nodes-Guides/assets/44331529/3b9399a8-88d5-4150-8e07-cd58291ddb16)

[WebSite](https://wardenprotocol.org/)\
[GitHub](https://github.com/warden-protocol/wardenprotocol)
=
[EXPLORER](https://explorer.stavr.tech/Warden-Testnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
wget -O wardent https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Warden/wardent && chmod +x wardent && ./wardent
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.21.4
```python
ver="1.21.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 12.03.24
```python
cd $HOME && mkdir -p go/bin/
git clone --depth 1 --branch v0.1.0 https://github.com/warden-protocol/wardenprotocol/
cd  wardenprotocol/warden/cmd/wardend
go build
sudo mv wardend $HOME/go/bin/

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`wardend version --long | grep -e version -e commit`
- version: 
- commit: 

```python
wardend init STAVR_guide
wardend config chain-id alfama
```    

## Create/recover wallet
```python
wardend keys add <walletname>
  OR
wardend keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.warden/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Warden/genesis.json"

```
`sha256sum $HOME/.warden/config/genesis.json`
+ f29ce94657e35706d7868bc725a3fbfd7530c1508842e6a339920a45d28e51b3

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uward\"/;" ~/.warden/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.warden/config/config.toml
peers="6a8de92a3bb422c10f764fe8b0ab32e1e334d0bd@sentry-1.alfama.wardenprotocol.org:26656,7560460b016ee0867cae5642adace5d011c6c0ae@sentry-2.alfama.wardenprotocol.org:26656,24ad598e2f3fc82630554d98418d26cc3edf28b9@sentry-3.alfama.wardenprotocol.org:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.warden/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.warden/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.warden/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.warden/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.warden/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.warden/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.warden/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Warden/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/wardend.service > /dev/null <<EOF
[Unit]
Description=wardend
After=network-online.target

[Service]
User=$USER
ExecStart=$(which wardend) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Warden Testnet
```python
SNAP_RPC="https://warden.rpc.t.stavr.tech:443"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.warden/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height) \
&& BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)) \
&& TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.warden/config/config.toml; \
wardend tendermint unsafe-reset-all --home $HOME/.warden
wget -O $HOME/.warden/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Warden/addrbook.json"
sudo systemctl restart wardend && journalctl -u wardend -f -o cat
```
# SnapShot Testnet updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop wardend
cp $HOME/.warden/data/priv_validator_state.json $HOME/.warden/priv_validator_state.json.backup
rm -rf $HOME/.warden/data
curl -o - -L https://warden-t.snapshot.stavr.tech/warden-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.warden --strip-components 2
mv $HOME/.warden/priv_validator_state.json.backup $HOME/.warden/data/priv_validator_state.json
wget -O $HOME/.warden/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Warden/addrbook.json"
sudo systemctl restart wardend && journalctl -u wardend -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable wardend
sudo systemctl restart wardend && sudo journalctl -u wardend -f -o cat
```

### Create validator
#pubkey
```python
wardend tendermint show-validator --home /root/.warden
```
```python
cd $HOME
nano validator.json
{
  "pubkey": {"#pubkey"},
  "amount": "1000000uward",
  "moniker": "STAVR_guide",
  "identity": "",
  "website": "",
  "security": "",
  "details": "",
  "commission-rate": "0.05",
  "commission-max-rate": "0.2",
  "commission-max-change-rate": "0.2",
  "min-self-delegation": "1"
}
```
```python
wardend --home $HOME/.warden tx staking create-validator $HOME/validator.json --from WalletName  --chain-id alfama --fees 500uward -y
```
[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Warden)
=


## Delete node
```python
sudo systemctl stop wardend
sudo systemctl disable wardend
rm /etc/systemd/system/wardend.service
sudo systemctl daemon-reload
cd $HOME
rm -rf wardenprotocol
rm -rf .wardend
rm -rf $(which wardend)
```
#
### Sync Info
```python
wardend status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
wardend status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u wardend -f -o cat
```
### Check Balance
```python
wardend query bank balances warden...addressjkl1yjgn7z09ua9vms259j
```


<h1 align="center"> 📚Useful commands📚 </h1>

# ⚙️Service

#### Info
```python
wardend status 2>&1 | jq .NodeInfo
wardend status 2>&1 | jq .SyncInfo
wardend status 2>&1 | jq .ValidatorInfo
```
#### Check node logs
```python
sudo journalctl -fu wardend -o cat
```
#### Check service status
```python
sudo systemctl status wardend
```
#### Restart service
```python
sudo systemctl restart wardend
```
#### Stop service
```python
sudo systemctl stop wardend
```
#### Start service
```python
sudo systemctl start wardend
```
#### reload/disable/enable
```python
sudo systemctl daemon-reload
sudo systemctl disable wardend
sudo systemctl enable wardend
```
#### Your Peer
```python
echo $(wardend tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.warden/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

# 🥅Working with keys

#### New Key or Recover Key
```python
wardend keys add Wallet_Name
      OR
wardend keys add Wallet_Name --recover
```
#### Check all keys
```python
wardend keys list
```
#### Check Balance
```python
wardend query bank balances warden...addressjkl1yjgn7z09ua9vms259j
```
#### Delete Key
```python
wardend keys delete Wallet_Name
```
#### Export Key
```python
wardend keys export wallet
```
#### Import Key
```python
wardend keys import wallet wallet.backup
```

# 🚀Validator Management

#### Edit Validator
```python
wardend tx staking edit-validator \
--new-moniker "Your_Moniker" \
--identity "Keybase_ID" \
--details "Your_Description" \
--website "Your_Website" \
--security-contact "Your_Email" \
--chain-id alfama \
--commission-rate 0.05 \
--from Wallet_Name \
--gas 350000 -y
```

#### Your Valoper-Address
```python
wardend keys show Wallet_Name --bech val
```
#### Your Valcons-Address
```python
wardend tendermint show-address
```
#### Your Validator-Info
```python
wardend query staking validator wardenvaloperaddress......
```
#### Jail Info
```python
wardend query slashing signing-info $(wardend tendermint show-validator)
```
#### Unjail
```python
wardend tx slashing unjail --from Wallet_name --chain-id alfama --gas 350000 -y
```
#### Active Validators List
```python
wardend q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Inactive Validators List
```python
wardend q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Check that your key matches the validator  (Win - Good.  Lose - Bad)
```python
VALOPER=Enter_Your_valoper_Here
[[ $(wardend  q staking validator $VALOPER -oj | jq -r .consensus_pubkey.key) = $(wardend status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\nYou win\n" || echo -e "\nYou lose\n"
```

#### Withdraw all rewards from all validators
```python
wardend tx distribution withdraw-all-rewards --from Wallet_Name --chain-id alfama --gas 350000 -y
```
#### Withdraw and commission from your Validator
```python
wardend tx distribution withdraw-rewards wardendvaloper1amxp0k0hg4edrxg85v07t9ka2tfuhamhldgf8e --from Wallet_Name --gas 350000 --chain-id=alfama --commission -y
```
#### Delegate tokens to your validator
```python
wardend tx staking delegate Your_wardenvalpoer........ "100000000"uward --from Wallet_Name --gas 350000 --chain-id=alfama -y
```
#### Delegate tokens to different validator
```python
wardend tx staking delegate wardenvalpoer........ "100000000"uward --from Wallet_Name --gas 350000 --chain-id=alfama -y
```
#### Redelegate tokens to another validator
```python
wardend tx staking redelegate Your_wardenvalpoer........ wardendvalpoer........ "100000000"uward --from Wallet_Name --gas 350000  --chain-id=alfama -y
```

#### Unbond tokens from your validator or different validator
```python
wardend tx staking unbond Your_wardenvalpoer........ "100000000"uward --from Wallet_Name --gas 350000 --chain-id=alfama -y
wardend tx staking unbond wardendvalpoer........ "100000000"uward --from Wallet_Name --gas 350000 --chain-id=alfama -y
```

#### Transfer tokens from wallet to wallet
```python
wardend tx bank send Your_wardenaddress............ wardenaddress........... "1000000000000000000"uward --gas 350000 --chain-id=alfama -y
```

# 📝Governance

#### View all proposals
```python
wardend query gov proposals
```

#### View specific proposal
```python
wardend query gov proposal 1
```

#### Vote yes
```python
wardend tx gov vote 1 yes --from Wallet_Name --gas 350000  --chain-id=alfama -y
```
#### Vote no
```python
wardend tx gov vote 1 no --from Wallet_Name --gas 350000  --chain-id=alfama -y
```
#### Vote abstain
```python
wardend tx gov vote 1 abstain --from Wallet_Name --gas 350000  --chain-id=alfama -y
```
#### Vote no_with_veto
```python
wardend tx gov vote 1 no_with_veto --from Wallet_Name --gas 350000  --chain-id=alfama -y
```
