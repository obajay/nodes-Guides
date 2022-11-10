# Neutron Testnet guide

![neutron](https://user-images.githubusercontent.com/44331529/201018597-cccdc09f-0f52-4cf9-af71-4d42ddf53e35.png)

[WebSite](https://neutron.org/)
=
[EXPLORER](https://explorer.stavr.tech/neutron-testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# 1) Auto_install script
```bash
SOOON
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

# Build 10.11.22
```bash
git clone -b v0.1.0 https://github.com/neutron-org/neutron.git
cd neutron
make install
```
`neutrond version --long`
- version: 0.1.0
- commit: a9e8ba5ebb9230bec97a4f2826d75a4e0e6130d9


```bash
neutrond init STAVRguide --chain-id quark-1

```    

## Create/recover wallet
```bash
neutrond keys add <walletname>
              or
neutrond keys add <walletname> --recover
```

## Download Genesis
```bash
curl -s https://raw.githubusercontent.com/neutron-org/testnets/main/quark/genesis.json > ~/.neutrond/config/genesis.json
```
`sha256sum $HOME/.neutrond/config/genesis.json`
+ 357c4d33fad26c001d086c0705793768ef32c884a6ba4aa73237ab03dd0cc2b4

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0untrn\"/" $HOME/.neutrond/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.neutrond/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.neutrond/config/config.toml
peers="fcde59cbba742b86de260730d54daa60467c91a5@23.109.158.180:26656,5bdc67a5d5219aeda3c743e04fdcd72dcb150ba3@65.109.31.114:2480,3e9656706c94ae8b11596e53656c80cf092abe5d@65.21.250.197:46656,9cb73281f6774e42176905e548c134fc45bbe579@162.55.134.54:26656,27b07238cf2ea76acabd5d84d396d447d72aa01b@65.109.54.15:51656,f10c2cb08f82225a7ef2367709e8ac427d61d1b5@57.128.144.247:26656,20b4f9207cdc9d0310399f848f057621f7251846@222.106.187.13:40006,5019864f233cee00f3a6974d9ccaac65caa83807@162.19.31.150:55256,2144ce0e9e08b2a30c132fbde52101b753df788d@194.163.168.99:26656,b37326e3acd60d4e0ea2e3223d00633605fb4f79@nebula.p2p.org:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.neutrond/config/config.toml
seeds="e2c07e8e6e808fb36cca0fc580e31216772841df@seed-1.quark.ntrn.info:26656,c89b8316f006075ad6ae37349220dd56796b92fa@tenderseed.ccvalidators.com:29001"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.neutrond/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.neutrond/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.neutrond/config/config.toml

```

Update your $HOME/.neutrond/config/config.toml and set `timeout_commit = "2s"`.

### Pruning (optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.neutrond/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.neutrond/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.neutrond/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.neutrond/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.neutrond/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.neutrond/config/addrbook.json "SOOON"
```

# Create a service file
```bash
sudo tee /etc/systemd/system/neutrond.service > /dev/null <<EOF
[Unit]
Description=neutron
After=network-online.target

[Service]
User=$USER
ExecStart=$(which neutrond) start
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
sudo systemctl enable neutrond
sudo systemctl restart neutrond && sudo journalctl -u neutrond -f -o cat
```

### Create validator
```bash
neutrond tx staking create-validator \
  --amount 1000000untrn \
  --from <walletName> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(neutrond tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id quark-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Delete node
```bash
sudo systemctl stop neutrond && \
sudo systemctl disable neutrond && \
rm /etc/systemd/system/neutrond.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf neutron && \
rm -rf .neutrond && \
rm -rf $(which neutrond)
```
#
### Sync Info
```bash
neutrond status 2>&1 | jq .SyncInfo
```
### NodeINfo
```bash
neutrond status 2>&1 | jq .NodeInfo
```
### Check node logs
```bash
sudo journalctl -u neutrond -f -o cat
```
### Check Balance
```bash
neutrond query bank balances neutron...address1yjgn7z09ua9vms259j
```
