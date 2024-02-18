# Side Testnet guide

![side](https://github.com/obajay/nodes-Guides/assets/44331529/40f14dda-75e9-4bd7-8d8a-8710f52dcf45)

[WebSite](https://side.one/)\
[GitHub](https://github.com/sideprotocol/sidechain.git)
=
[EXPLORER](https://explorer.stavr.tech/Side-Testnet)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python
SOOON
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
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

# Build 18.02.24
```python
cd $HOME && mkdir -p go/bin/
git clone -b dev https://github.com/sideprotocol/sidechain.git
cd sidechain
git checkout v0.0.3
make install

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`sided version --long | grep -e commit -e version`
- version: 0.0.3
- commit: c364426fb41dacf2cdd05ca4d95e713a7292a93c

```python
sided init STAVR_guide --chain-id side-testnet-1
sided config chain-id side-testnet-1
```    

## Create/recover wallet
```python
sided keys add <walletname>
  OR
sided keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/sideprotocol/testnet/main/shambhala/genesis.json -O $HOME/.side/config/genesis.json

```
`sha256sum $HOME/.side/config/genesis.json`
+ 69bb55baeed046704d09f7590d8969dfafcc5f6afbd00d9479bd17ced44b7708

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uside\"/;" ~/.side/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.side/config/config.toml
peers="2eba9c8e6fb9d56bbdd10d007a598541c37f6493@13.212.61.41:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.side/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.side/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.side/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.side/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.side/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.side/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.side/config/addrbook.json "SOON"
```

# Create a service file
```python
sudo tee /etc/systemd/system/sided.service > /dev/null <<EOF
[Unit]
Description=sided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Side Testnet
```python
SOOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
SOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable sided
sudo systemctl restart sided && sudo journalctl -u sided -f -o cat
```

### Create validator
```python
sided tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000uside \
--pubkey $(sided tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id side-testnet-1 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Side)
=


## Delete node
```python
sudo systemctl stop sided
sudo systemctl disable sided
rm /etc/systemd/system/sided.service
sudo systemctl daemon-reload
cd $HOME
rm -rf sidechain
rm -rf .side
rm -rf $(which sided)
```
#
### Sync Info
```python
sided status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
sided status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u sided -f -o cat
```
### Check Balance
```python
sided query bank balances sided...addressjkl1yjgn7z09ua9vms259j
```


<h1 align="center"> 📚Useful commands📚 </h1>

# ⚙️Service

#### Info
```python
sided status 2>&1 | jq .NodeInfo
sided status 2>&1 | jq .SyncInfo
sided status 2>&1 | jq .ValidatorInfo
```
#### Check node logs
```python
sudo journalctl -fu sided -o cat
```
#### Check service status
```python
sudo systemctl status sided
```
#### Restart service
```python
sudo systemctl restart sided
```
#### Stop service
```python
sudo systemctl stop sided
```
#### Start service
```python
sudo systemctl start sided
```
#### reload/disable/enable
```python
sudo systemctl daemon-reload
sudo systemctl disable sided
sudo systemctl enable sided
```
#### Your Peer
```python
echo $(sided tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.side/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

# 🥅Working with keys

#### New Key or Recover Key
```python
sided keys add Wallet_Name
      OR
sided keys add Wallet_Name --recover
```
#### Check all keys
```python
sided keys list
```
#### Check Balance
```python
sided query bank balances sided...addressjkl1yjgn7z09ua9vms259j
```
#### Delete Key
```python
sided keys delete Wallet_Name
```
#### Export Key
```python
sided keys export wallet
```
#### Import Key
```python
sided keys import wallet wallet.backup
```

# 🚀Validator Management

#### Edit Validator
```python
sided tx staking edit-validator \
--new-moniker "Your_Moniker" \
--identity "Keybase_ID" \
--details "Your_Description" \
--website "Your_Website" \
--security-contact "Your_Email" \
--chain-id side-testnet-1 \
--commission-rate 0.05 \
--from Wallet_Name \
--gas 350000 -y
```

#### Your Valoper-Address
```python
sided keys show Wallet_Name --bech val
```
#### Your Valcons-Address
```python
sided tendermint show-address
```
#### Your Validator-Info
```python
sided query staking validator sidedvaloperaddress......
```
#### Jail Info
```python
sided query slashing signing-info $(sided tendermint show-validator)
```
#### Unjail
```python
sided tx slashing unjail --from Wallet_name --chain-id side-testnet-1 --gas 350000 -y
```
#### Active Validators List
```python
sided q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Inactive Validators List
```python
sided q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Check that your key matches the validator  (Win - Good.  Lose - Bad)
```python
VALOPER=Enter_Your_valoper_Here
[[ $(sided  q staking validator $VALOPER -oj | jq -r .consensus_pubkey.key) = $(sided status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\nYou win\n" || echo -e "\nYou lose\n"
```

#### Withdraw all rewards from all validators
```python
sided tx distribution withdraw-all-rewards --from Wallet_Name --chain-id side-testnet-1 --gas 350000 -y
```
#### Withdraw and commission from your Validator
```python
sided tx distribution withdraw-rewards sidedvaloper1a........ --from Wallet_Name --gas 350000 --chain-id=side-testnet-1 --commission -y
```
#### Delegate tokens to your validator
```python
sided tx staking delegate Your_sidedvalpoer........ "100000000"uside --from Wallet_Name --gas 350000 --chain-id=side-testnet-1 -y
```
#### Delegate tokens to different validator
```python
sided tx staking delegate sidedvalpoer........ "100000000"uside --from Wallet_Name --gas 350000 --chain-id=side-testnet-1 -y
```
#### Redelegate tokens to another validator
```python
sided tx staking redelegate Your_sidedvalpoer........ sidedvalpoer........ "100000000"uside --from Wallet_Name --gas 350000  --chain-id=side-testnet-1 -y
```

#### Unbond tokens from your validator or different validator
```python
sided tx staking unbond Your_sidedvalpoer........ "100000000"uside --from Wallet_Name --gas 350000 --chain-id=side-testnet-1 -y
sided tx staking unbond sidedvalpoer........ "100000000"uside --from Wallet_Name --gas 350000 --chain-id=side-testnet-1 -y
```

#### Transfer tokens from wallet to wallet
```python
sided tx bank send Your_sidedaddress............ sidedaddress........... "1000000000000000000"uside --gas 350000 --chain-id=side-testnet-1 -y
```

# 📝Governance

#### View all proposals
```python
sided query gov proposals
```

#### View specific proposal
```python
sided query gov proposal 1
```

#### Vote yes
```python
sided tx gov vote 1 yes --from Wallet_Name --gas 350000  --chain-id=side-testnet-1 -y
```
#### Vote no
```python
sided tx gov vote 1 no --from Wallet_Name --gas 350000  --chain-id=side-testnet-1 -y
```
#### Vote abstain
```python
sided tx gov vote 1 abstain --from Wallet_Name --gas 350000  --chain-id=side-testnet-1 -y
```
#### Vote no_with_veto
```python
sided tx gov vote 1 no_with_veto --from Wallet_Name --gas 350000  --chain-id=side-testnet-1 -y
```