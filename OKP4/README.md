# OKP4 Testnet guide

![Okp4](https://user-images.githubusercontent.com/44331529/197152847-749c938c-c385-4698-bfa5-3f159297f391.png)

[WEBSITE](https://okp4.network/) \
[GitHub](https://github.com/okp4)
=
[EXPLORER 1](https://explorer.stavr.tech/okp4/staking) \
[EXPLORER 2](https://explorer.bccnodes.com/okp4/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Tetsnet   |   4|  8GB | 100GB    |


# 1) Auto_install script
```bash
soon
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

# Build 10.10.22
```bash
cd $HOME
git clone https://github.com/okp4/okp4d.git
cd okp4d
make install
```
`okp4d version`
- version: 2.2.0

```bash
okp4d init STAVRguide --chain-id okp4-nemeton
```    

## Create/recover wallet
```bash
okp4d keys add <walletname>
okp4d keys add <walletname> --recover
```

## Download Genesis
```bash
wget -qO $HOME/.okp4d/config/genesis.json "https://raw.githubusercontent.com/okp4/networks/main/chains/nemeton/genesis.json"
```
`sha256sum $HOME/.okp4d/config/genesis.json`
+ c2e8fff161850e419e1cb1bef3648c0ed0db961b7713151f10f2509e3fc2ff40

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uknow\"/;" ~/.okp4d/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.okp4d/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.okp4d/config/config.toml
peers="f595a1386d5ca2e0d2cd81d3c6372c3bf84bbd16@65.109.31.114:2280,a49302f8999e5a953ebae431c4dde93479e17155@162.19.71.91:26656,dc14197ed45e84ca3afb5428eb04ea3097894d69@88.99.143.105:26656,79d179ea2e1fbdcc0c59a95ab7f1a0c48438a693@65.108.106.131:26706,501ad80236a5ac0d37aafa934c6ec69554ce7205@89.149.218.20:26656,5fbddca54548bf125ee96bb388610fe1206f087f@51.159.66.123:26656,769f74d3bb149216d0ab771d7767bd39585bc027@185.196.21.99:26656,024a57c0bb6d868186b6f627773bf427ec441ab5@65.108.2.41:36656,fff0a8c202befd9459ff93783a0e7756da305fe3@38.242.150.63:16656,2bfd405e8f0f176428e2127f98b5ec53164ae1f0@142.132.149.118:26656,bf5802cfd8688e84ac9a8358a090e99b5b769047@135.181.176.109:53656,dc9a10f2589dd9cb37918ba561e6280a3ba81b76@54.244.24.231:26656,085cf43f463fe477e6198da0108b0ab08c70c8ab@65.108.75.237:6040,803422dc38606dd62017d433e4cbbd65edd6089d@51.15.143.254:26656,b8330b2cb0b6d6d8751341753386afce9472bac7@89.163.208.12:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.okp4d/config/config.toml
seeds=""
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
wget -O $HOME/.mande-chain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Mande%20Chain/addrbook.json"
```

# StateSync
```bash
SNAP_RPC=http://38.242.199.93:24657
peers="a3e3e20528604b26b792055be84e3fd4de70533b@38.242.199.93:24656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.mande-chain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.mande-chain/config/config.toml

mande-chaind tendermint unsafe-reset-all --home /root/.mande-chain --keep-addr-book
systemctl restart mande-chaind && journalctl -u mande-chaind -f -o cat

```

# Create a service file
```bash
sudo tee /etc/systemd/system/mande-chaind.service > /dev/null <<EOF
[Unit]
Description=mande-chaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which mande-chaind) start
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
sudo systemctl enable mande-chaind
sudo systemctl restart mande-chaind && sudo journalctl -u mande-chaind -f -o cat
```
## Use faucet
```bash
curl -d '{"address":"YOURWALLETADDRESS"}' -H 'Content-Type: application/json' http://35.224.207.121:8080/request
```

### Create validator
```bash
mande-chaind tx staking create-validator \
--chain-id mande-testnet-1 \
--amount 0cred \
--pubkey "$(mande-chaind tendermint show-validator)" \
--from <wallet> \
--moniker="STAVRguide" \
--fees 1000mand
```

## Delete node
```bash
sudo systemctl stop mande-chaind && \
sudo systemctl disable mande-chaind && \
rm /etc/systemd/system/mande-chaind.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .mande-chain && \
rm -rf $(which mande-chaind)
```
