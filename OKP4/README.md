# OKP4 Testnet guide

![Okp4](https://user-images.githubusercontent.com/44331529/197152847-749c938c-c385-4698-bfa5-3f159297f391.png)

[WEBSITE](https://okp4.network/) \
[GitHub](https://github.com/okp4)
=
[EXPLORER 1](https://explorer.stavr.tech/okp4-testnet/staking) \
[EXPLORER 2](https://explorer.bccnodes.com/okp4/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Tetsnet   |   4|  8GB | 100GB    |


# 1) Auto_install script
```bash
wget -O okp https://raw.githubusercontent.com/obajay/nodes-Guides/main/OKP4/okp && chmod +x okp && ./okp
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

# Build 01.12.22
```bash
cd $HOME
git clone https://github.com/okp4/okp4d.git
cd okp4d
git checkout v3.0.0
make install
```
`okp4d version`
- version: 3.0.0

```bash
okp4d init STAVRguide --chain-id okp4-nemeton-1
okp4d config chain-id okp4-nemeton-1
```    

## Create/recover wallet
```bash
okp4d keys add <walletname>
okp4d keys add <walletname> --recover
```

## Download Genesis
```bash
curl -Ls https://snapshots.kjnodes.com/okp4-testnet/genesis.json > $HOME/.okp4d/config/genesis.json
```
`sha256sum $HOME/.okp4d/config/genesis.json`
+ 2ec25f81cc2abecbc0da3de45b052ea3314d0d658b1b7f4c7b6a48d09254c742

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uknow\"/;" ~/.okp4d/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.okp4d/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.okp4d/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.okp4d/config/config.toml
seeds="3f472746f46493309650e5a033076689996c8881@okp4-testnet.rpc.kjnodes.com:36659"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.okp4d/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.okp4d/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.okp4d/config/config.toml

```
### Pruning (optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.okp4d/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.okp4d/config/config.toml
```

## Download addrbook
```bash
curl -Ls https://snapshots.kjnodes.com/okp4-testnet/addrbook.json > $HOME/.okp4d/config/addrbook.json
```


# Create a service file
```bash
sudo tee /etc/systemd/system/okp4d.service > /dev/null <<EOF
[Unit]
Description=okp4d
After=network-online.target

[Service]
User=$USER
ExecStart=$(which okp4d) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```bash
sudo systemctl daemon-reload
sudo systemctl enable okp4d
sudo systemctl restart okp4d && sudo journalctl -u okp4d -f -o cat
```

### Create validator
```bash
okp4d tx staking create-validator \
  --amount 1000000uknow \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(okp4d tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id okp4-nemeton-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop okp4d && \
sudo systemctl disable okp4d && \
rm /etc/systemd/system/okp4d.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf okp4d && \
rm -rf .okp4d && \
rm -rf $(which okp4d)
```
