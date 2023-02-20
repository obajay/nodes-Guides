# Uptick testnet guide

![Uptick](https://user-images.githubusercontent.com/44331529/180614523-9a7e76e9-9243-4f38-8938-1cdaa13e2cf6.png)

[Website](https://uptick.network/ ) \
[EXPLORER](https://explorer.stavr.tech/uptick/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4| 8GB  | 150GB    |

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

# Build 20.02.23
```python
cd $HOME
git clone https://github.com/UptickNetwork/uptick.git
cd uptick
git checkout v0.2.5
make install
```
*******🟢UPDATE🟢******* 20.02.23
```python
cd $HOME/uptick
git fetch --all
git checkout v0.2.5
make install
sudo systemctl restart uptickd && journalctl -u uptickd -f -o cat
```

`uptickd version`
+ version: v0.2.5
+ commit: 17716164d170aa3bd1ca386f1216f101f3f60c5c

## Initialization
```python
uptickd init STAVRguide --chain-id uptick_7000-2
uptickd config chain-id uptick_7000-2
```

## Create/recover wallet
```python
uptickd keys add <walletname>
uptickd keys add <walletname> --recover
```

## Genesis
```python
curl -o $HOME/.uptickd/config/genesis.json https://raw.githubusercontent.com/UptickNetwork/uptick-testnet/main/uptick_7000-2/genesis.json
```

## Peers/Seeds/Gas
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0auptick\"/;" ~/.uptickd/config/app.toml
external_address=$(wget -qO- eth0.me)
peers="eecdfb17919e59f36e5ae6cec2c98eeeac05c0f2@peer0.testnet.uptick.network:26656,178727600b61c055d9b594995e845ee9af08aa72@peer1.testnet.uptick.network:26656,f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@uptick-seed.p2p.brocha.in:30554,94b63fddfc78230f51aeb7ac34b9fb86bd042a77@uptick-testnet-rpc.p2p.brocha.in:30556,902a93963c96589432ee3206944cdba392ae5c2d@65.108.42.105:27656"
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

## SNAPSHOT
```python
wget https://download.uptick.network/download/uptick/testnet/node/data/data.tar.gz
tar -C $HOME/.uptickd/data/ -zxvf data.tar.gz --strip-components 1
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.uptickd/config/app.toml
sudo systemctl restart uptickd && sudo journalctl -u uptickd -f -o cat
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
--chain-id uptick_7000-2 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.1 \
--min-self-delegation="1" \
--amount=1000000auptick \
--pubkey $(uptickd tendermint show-validator) \
--moniker "STAVRguide" \
--from=<name_wallet> \
--gas="auto" \
--fees 555auptick
```

## Delete node
```python
sudo systemctl stop uptickd && \
sudo systemctl disable uptickd && \
rm /etc/systemd/system/uptickd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .uptickd && \
rm -rf uptick-v0.2.3 && \
rm -rf $(which uptickd)
```
