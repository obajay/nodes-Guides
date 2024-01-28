[🔥OUR VALIDATOR🔥](https://restake.app/uptick/uptickvaloper1n9urj4d6mngtuhpfysdxu7nq72e8830wkx5mug)
=

# Uptick Mainnet guide
![Uptick](https://user-images.githubusercontent.com/44331529/180614523-9a7e76e9-9243-4f38-8938-1cdaa13e2cf6.png)

[Website](https://uptick.network/ ) \
[EXPLORER 1](https://explorer.stavr.tech/Uptick-Mainnet/staking) \
[EXPLORER 2](https://exp.utsa.tech/uptick)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 250GB    |

# 1) Auto_install script
```python
wget -O uptickm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Uptick/uptickm && chmod +x uptickm && ./uptickm
```

# 2) Manual installation
### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```


## GO 19.4 (one command)
```python
cd $HOME && \
ver="1.19.4" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 29.01.24
```python
cd $HOME
git clone https://github.com/UptickNetwork/uptick.git
cd uptick
git checkout v0.2.17
make install
```
*******🟢UPDATE🟢******* 29.01.24
```python
cd $HOME/uptick
git fetch --all
git checkout v0.2.17
make install
uptickd version --long | grep -e commit -e version
#commit: b4b58f20eb48234ba3996456158f380419da11cf
#version: v0.2.17
sudo systemctl restart uptickd && journalctl -u uptickd -f -o cat
```

`uptickd version --long`
+ version: v0.2.17
+ commit: b4b58f20eb48234ba3996456158f380419da11cf

## Initialization
```python
uptickd init STAVR_guide --chain-id uptick_117-1
uptickd config chain-id uptick_117-1
```

## Create/recover wallet
```python
uptickd keys add <walletname>
uptickd keys add <walletname> --recover
```

## Genesis
```python
wget -O $HOME/.uptickd/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Uptick/genesis.json"
```

## Peers/Seeds/Gas
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0auptick\"/;" ~/.uptickd/config/app.toml
external_address=$(wget -qO- eth0.me)
peers="170397e75ca2b0f4e9f3b1bb5d0d23f9b10f01c7@uptick-sentry-1.p2p.brocha.in:30597,c0b33353fb70d8d71dcb9c8848b3b4207bd56951@uptick-sentry-2.p2p.brocha.in:30598,23e76540bea9b6851b92e280d7e0c123a0d49521@uptick-sentry-3.p2p.brocha.in:30599,94b63fddfc78230f51aeb7ac34b9fb86bd042a77@uptick-rpc.p2p.brocha.in:30601,f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@uptick.seed.brocha.in:30600,48e7e8ca23b636f124e70092f4ba93f98606f604@54.37.129.164:55056"
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.uptickd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.uptickd/config/config.toml
```

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.uptickd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.uptickd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.uptickd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.uptickd/config/app.toml
```

### Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.uptickd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.uptickd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Uptick/addrbook.json"
```
## StateSync Uptick Mainnet
```python
SNAP_RPC=http://uptick.rpc.m.stavr.tech:3157
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.uptickd/config/config.toml
uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.uptickd/config/app.toml
sudo systemctl restart uptickd && journalctl -u uptickd -f -o cat
```

## SnapShot Mainnet (~0.2GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop uptickd
cp $HOME/.uptickd/data/priv_validator_state.json $HOME/.uptickd/priv_validator_state.json.backup
rm -rf $HOME/.uptickd/data
curl -o - -L http://uptick.snapshot.stavr.tech:1027/uptickd/uptickd-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.uptickd --strip-components 2
mv $HOME/.uptickd/priv_validator_state.json.backup $HOME/.uptickd/data/priv_validator_state.json
wget -O $HOME/.uptickd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Uptick/addrbook.json"
sudo systemctl restart uptickd && journalctl -u uptickd -f -o cat
```

# Create a service file
```python
sudo tee /etc/systemd/system/uptickd.service > /dev/null <<EOF
[Unit]
Description=uptick
After=network-online.target

[Service]
User=$USER
ExecStart=$(which uptickd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload && \
sudo systemctl enable uptickd && \
sudo systemctl restart uptickd && sudo journalctl -u uptickd -f -o cat
```

## Create validator
```python
uptickd tx staking create-validator \
--chain-id uptick_117-1 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.1 \
--min-self-delegation="1" \
--amount=1000000000000000000auptick \
--pubkey $(uptickd tendermint show-validator) \
--moniker "STAVR_guide" \
--from=<name_wallet> \
--gas="auto" \
--fees 555auptick
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Uptick)
=

## Delete node
```python
sudo systemctl stop uptickd
sudo systemctl disable uptickd
rm /etc/systemd/system/uptickd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .uptickd
rm -rf uptick
rm -rf $(which uptickd)
```
