# Althea Mainnet guide

![Althea_Logo-BLUE_SIGNAL](https://user-images.githubusercontent.com/44331529/218240936-c2095305-1a28-45f6-8ccd-d068a4fe5754.svg)

[WebSite](https://www.althea.net/)\
[GitHub](https://github.com/althea-net)
=
[EXPLORER](https://explorer.stavr.tech/Althea-testnetL1/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB    |


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

# Build 06.03.24
```python
cd $HOME && mkdir -p go/bin/
wget https://github.com/althea-net/althea-L1/releases/download/v1.0.0-rc1/althea-linux-amd64
chmod +x althea-linux-amd64
sudo mv althea-linux-amd64 $HOME/go/bin/althea

```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`althea version --long | head`
- version: v1.0.0-rc1
- commit: 92f53e521ec93807d75769bcef3d8ed2dd7a11c8

```python
althea init STAVR_guide --chain-id althea_417834-4
althea config chain-id althea_417834-4
```    

## Create/recover wallet
```python
althea keys add <walletname>
  OR
althea keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.althea/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Althea/genesis.json"

```
`sha256sum $HOME/.althea/config/genesis.json`
+ fec9467984e4065dae78694f75006f27a1338cae9dac43dca2cec4e8f403bfb8

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0aalthea\"/;" ~/.althea/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.althea/config/config.toml
peers="ab9a9e6ea747839652dfe4480e66a5eb78a385e8@51.81.167.60:17200,cbdcc6edc9b2cbd652fe94ef774e1f483095a8a3@66.172.36.142:14656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.althea/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.althea/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.althea/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.althea/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.althea/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.althea/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.althea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Althea/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/althea.service > /dev/null <<EOF
[Unit]
Description=althea
After=network-online.target

[Service]
User=$USER
ExecStart=$(which althea) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Althea Testnet
```python
SNAP_RPC=https://althea.rpc.m.stavr.tech:443
peers="a1ef55814e2b9aa6c75fbdda52a0ce3d10aebfec@althea.peers.stavr.tech:17886"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.althea/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.althea/config/config.toml
althea tendermint unsafe-reset-all --home /root/.althea
systemctl restart althea && journalctl -u althea -f -o cat
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop althea
cp $HOME/.althea/data/priv_validator_state.json $HOME/.althea/priv_validator_state.json.backup
rm -rf $HOME/.althea/data
curl -o - -L http://althea.snapshot.stavr.tech:1020/althea/althea-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.althea --strip-components 2
mv $HOME/.althea/priv_validator_state.json.backup $HOME/.althea/data/priv_validator_state.json
wget -O $HOME/.althea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Althea/addrbook.json"
sudo systemctl restart althea && journalctl -u althea -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable althea
sudo systemctl restart althea && sudo journalctl -u althea -f -o cat
```

### Create validator
```python
althea tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000000000000000aalthea \
--pubkey $(althea tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id althea_417834-4 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Althea)
=


## Delete node
```python
sudo systemctl stop althea
sudo systemctl disable althea
rm /etc/systemd/system/althea.service
sudo systemctl daemon-reload
cd $HOME
rm -rf althea-chain
rm -rf .althea
rm -rf $(which althea)
```
#
### Sync Info
```python
althea status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
althea status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u althea -f -o cat
```
### Check Balance
```python
althea query bank balances althea...addressjkl1yjgn7z09ua9vms259j
```


<h1 align="center"> 📚Useful commands📚 </h1>

# ⚙️Service

#### Info
```python
althea status 2>&1 | jq .NodeInfo
althea status 2>&1 | jq .SyncInfo
althea status 2>&1 | jq .ValidatorInfo
```
#### Check node logs
```python
sudo journalctl -fu althea -o cat
```
#### Check service status
```python
sudo systemctl status althea
```
#### Restart service
```python
sudo systemctl restart althea
```
#### Stop service
```python
sudo systemctl stop althea
```
#### Start service
```python
sudo systemctl start althea
```
#### reload/disable/enable
```python
sudo systemctl daemon-reload
sudo systemctl disable althea
sudo systemctl enable althea
```
#### Your Peer
```python
echo $(althea tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.althea/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

# 🥅Working with keys

#### New Key or Recover Key
```python
althea keys add Wallet_Name
      OR
althea keys add Wallet_Name --recover
```
#### Check all keys
```python
althea keys list
```
#### Check Balance
```python
althea query bank balances althea...addressjkl1yjgn7z09ua9vms259j
```
#### Delete Key
```python
althea keys delete Wallet_Name
```
#### Export Key
```python
althea keys export wallet
```
#### Import Key
```python
althea keys import wallet wallet.backup
```

# 🚀Validator Management

#### Edit Validator
```python
althea tx staking edit-validator \
--new-moniker "Your_Moniker" \
--identity "Keybase_ID" \
--details "Your_Description" \
--website "Your_Website" \
--security-contact "Your_Email" \
--chain-id althea_417834-4 \
--commission-rate 0.05 \
--from Wallet_Name \
--gas 350000 -y
```

#### Your Valoper-Address
```python
althea keys show Wallet_Name --bech val
```
#### Your Valcons-Address
```python
althea tendermint show-address
```
#### Your Validator-Info
```python
althea query staking validator altheavaloperaddress......
```
#### Jail Info
```python
althea query slashing signing-info $(althea tendermint show-validator)
```
#### Unjail
```python
althea tx slashing unjail --from Wallet_name --chain-id althea_417834-4 --gas 350000 -y
```
#### Active Validators List
```python
althea q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Inactive Validators List
```python
althea q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Check that your key matches the validator  (Win - Good.  Lose - Bad)
```python
VALOPER=Enter_Your_valoper_Here
[[ $(althea  q staking validator $VALOPER -oj | jq -r .consensus_pubkey.key) = $(althea status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\nYou win\n" || echo -e "\nYou lose\n"
```

#### Withdraw all rewards from all validators
```python
althea tx distribution withdraw-all-rewards --from Wallet_Name --chain-id althea_417834-4 --gas 350000 -y
```
#### Withdraw and commission from your Validator
```python
althea tx distribution withdraw-rewards altheavaloper1amxp0k0hg4edrxg85v07t9ka2tfuhamhldgf8e --from Wallet_Name --gas 350000 --chain-id=althea_417834-4 --commission -y
```
#### Delegate tokens to your validator
```python
althea tx staking delegate Your_altheavalpoer........ "100000000"aalthea --from Wallet_Name --gas 350000 --chain-id=althea_417834-4 -y
```
#### Delegate tokens to different validator
```python
althea tx staking delegate altheavalpoer........ "100000000"aalthea --from Wallet_Name --gas 350000 --chain-id=althea_417834-4 -y
```
#### Redelegate tokens to another validator
```python
althea tx staking redelegate Your_altheavalpoer........ altheavalpoer........ "100000000"aalthea --from Wallet_Name --gas 350000  --chain-id=althea_417834-4 -y
```

#### Unbond tokens from your validator or different validator
```python
althea tx staking unbond Your_altheavalpoer........ "100000000"aalthea --from Wallet_Name --gas 350000 --chain-id=althea_417834-4 -y
althea tx staking unbond altheavalpoer........ "100000000"aalthea --from Wallet_Name --gas 350000 --chain-id=althea_417834-4 -y
```

#### Transfer tokens from wallet to wallet
```python
althea tx bank send Your_altheaaddress............ altheaaddress........... "1000000000000000000"aalthea --gas 350000 --chain-id=althea_417834-4 -y
```

# 📝Governance

#### View all proposals
```python
althea query gov proposals
```

#### View specific proposal
```python
althea query gov proposal 1
```

#### Vote yes
```python
althea tx gov vote 1 yes --from Wallet_Name --gas 350000  --chain-id=althea_417834-4 -y
```
#### Vote no
```python
althea tx gov vote 1 no --from Wallet_Name --gas 350000  --chain-id=althea_417834-4 -y
```
#### Vote abstain
```python
althea tx gov vote 1 abstain --from Wallet_Name --gas 350000  --chain-id=althea_417834-4 -y
```
#### Vote no_with_veto
```python
althea tx gov vote 1 no_with_veto --from Wallet_Name --gas 350000  --chain-id=althea_417834-3 -y
```
