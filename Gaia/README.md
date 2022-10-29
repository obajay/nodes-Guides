# Cosmos Testnet guide

![comsos](https://user-images.githubusercontent.com/44331529/198814368-03e53f68-0ef3-43e3-a070-f094582e7171.png)

[WEBSITE](https://cosmos.network/) \
[GitHub](https://github.com/cosmos/testnets/tree/master/public)
=
[EXPLORER](https://explorer.theta-testnet.polypore.xyz/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```bash
SOON
```

# 2) Manual installation

### Preparing the server

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19

```bash
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 15.10.22
```bash
git clone https://github.com/cosmos/gaia
cd gaia
git checkout v7.0.2 
make install
```
`gaiad version`
- version: v7.0.2

```bash
gaiad init STAVRguide --chain-id=theta-testnet-001
```    

## Create/recover wallet
```bash
gaiad keys add <walletname>
gaiad keys add <walletname> --recover
```

## Download Genesis
```bash
cd $HOME/.gaia/config
wget https://github.com/cosmos/testnets/blob/master/public/genesis.json.gz
tar -xvzf genesis.json.gz
rm -rf genesis.json.gz
```
`sha256sum $HOME/.gaia/config/genesis.json`
+ 522d7e5227ca35ec9bbee5ab3fe9d43b61752c6bdbb9e7996b38307d7362bb7e

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0stake\"/;" ~/.gaia/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.gaia/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.gaia/config/config.toml
peers="4308b2f0dba139e3238b40c480ef65f0966c787b@54.39.243.226:20656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.gaia/config/config.toml
seeds="https://seed-01.theta-testnet.polypore.xyz,https://seed-02.theta-testnet.polypore.xyz"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.gaia/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.gaia/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.gaia/config/config.toml

```
### Pruning (optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gaia/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gaia/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.gaia/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.gaia/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.gaia/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.gaia/config/addrbook.json "SOON"
```

# Create a service file
```bash
sudo tee /etc/systemd/system/gaiad.service > /dev/null << EOF
[Unit]
Description=Gaia
After=network-online.target
[Service]
User=$USER
ExecStart=$(which gaiad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

## Start
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable gaiad && \
sudo systemctl restart gaiad && sudo journalctl -u gaiad -f -o cat
```

### Create validator
```bash
gaiad tx staking create-validator \
  --amount 1000000uatom \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(gaiad tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id theta-testnet-001 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop gaiad && \
sudo systemctl disable gaiad && \
rm /etc/systemd/system/gaiad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf gaia && \
rm -rf .gaia && \
rm -rf $(which gaiad)
```
#
### Sync Info
```bash
gaiad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```bash
gaiad status 2>&1 | jq .NodeInfo
```
### Check node logs
```bash
sudo journalctl -u gaiad -f -o cat
```
### Check Balance
```bash
gaiad query bank balances cosmos...addresscosmos1yjgn7z09ua9vms259j
```
