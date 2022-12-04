# Sge Testnet guide

![sge](https://user-images.githubusercontent.com/44331529/203624604-ec312821-11cd-404e-8647-bbbcfddcaf8a.png)

[WebSite](https://sgenetwork.io/)
=
[EXPLORER 1](https://explorer.stavr.tech/sge-testnet/staking) \
[EXPLORER 2](https://exp.nodeist.net/t-sge/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```python

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

# Build 23.11.22
```python
git clone https://github.com/sge-network/sge
git clone https://github.com/sge-network/networks
cd sge
git fetch --tags
git checkout v0.0.3
cd sge
go mod tidy
make install
```
`sged version --long`
- version: v0.0.3
- commit: c2f074f15fa895b0d8e67a9d88bfd2b9d9833b2f

```python
sged init STAVRguide --chain-id sge-testnet-1
```    

## Create/recover wallet
```python
sged keys add <walletname>
sged keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.sge/config/genesis.json "https://raw.githubusercontent.com/sge-network/networks/master/sge-testnet-1/genesis.json"

```
`sha256sum $HOME/.sge/config/genesis.json`
+ 9a36a608e66fb194f404284c2d65aa5a7eb422b3efbd50869f270dfe65713e5b

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usge\"/" $HOME/.sge/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sge/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sge/config/config.toml
peers="27f0b281ea7f4c3db01fdb9f4cf7cc910ad240a6@209.34.206.44:26656,5f3196f370fa865bfd3e4a0653dc7853f613aba6@[2a01:4f9:1a:a718::10]:26656,afa90de6a195a4a2993b2501f12a1cd306f01d02@136.243.103.32:60856,dc75f5d2f9458767f39f62bd7eab3f499fdf2761@104.248.236.171:26656,1168931936c638e92ea6d93e2271b3fe5faee6d1@51.91.145.100:26656,8a7d722dba88326ee69fcc23b5b2ac93e36d7ff2@65.108.225.158:17756,445506c736895336e36dd4f8228a60c257b30e61@20.12.75.0:26656,971643c5b9f9d279cfb7ac1b14accd109231236b@65.108.15.170:26656,788bb7ee73c023f70c41360e9014544b12fe23f9@3.15.209.96:26656,26f0965f8cd53f2b3adc26f8ca5e893929b66c15@52.44.14.245:26656,4a3f59e30cde63d00aed8c3d15bef46b34ec2c7f@50.19.180.153:26656,31d742df5a427e241d1a6b1b22813c9cb4888c07@65.21.181.169:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sge/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sge/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.sge/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.sge/config/config.toml

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
wget -O $HOME/.sge/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/SGE/addrbook.json"
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
  --moniker STAVRguide \
  --chain-id sge-testnet-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop sged && \
sudo systemctl disable sged && \
rm /etc/systemd/system/sged.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf sge && \
rm -rf .sge && \
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
