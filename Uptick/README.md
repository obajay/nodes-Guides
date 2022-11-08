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
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 18.3 (one command)
```bash
ver="1.18.3" && \
cd $HOME && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 08.11.22
```bash
cd $HOME
wget https://github.com/UptickNetwork/uptick/releases/download/v0.2.4/uptick-linux-amd64-v0.2.4.tar.gz
tar -zxvf uptick-linux-amd64-v0.2.4.tar.gz
chmod +x $HOME/uptick-linux-amd64-v0.2.4/uptickd
mv $HOME/uptick-linux-amd64-v0.2.4/uptickd $HOME/go/bin/
```

`uptickd version`
+ 0.2.4

## Initialization
```bash
uptickd init STAVRguide --chain-id uptick_7000-1
uptickd config chain-id uptick_7000-1
```

## Create/recover wallet
```bash
uptickd keys add <walletname>
uptickd keys add <walletname> --recover
```

## Genesis
```bash
wget -O $HOME/.uptickd/config/genesis.json "https://raw.githubusercontent.com/UptickNetwork/uptick-testnet/main/uptick_7000-1/genesis.json"
```

## Peers/Seeds/Gas
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0auptick\"/;" ~/.uptickd/config/app.toml
external_address=$(wget -qO- eth0.me)
peers="eecdfb17919e59f36e5ae6cec2c98eeeac05c0f2@peer0.testnet.uptick.network:26656,178727600b61c055d9b594995e845ee9af08aa72@peer1.testnet.uptick.network:26656"
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.uptickd/config/config.toml
seeds="61f9e5839cd2c56610af3edd8c3e769502a3a439@seed0.testnet.uptick.network:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.uptickd/config/config.toml
```

### Pruning (optional)
```bash
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
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.uptickd/config/config.toml
```

## SNAPSHOT
```bash
wget https://download.uptick.network/download/uptick/testnet/node/data/data.tar.gz
tar -C $HOME/.uptickd/data/ -zxvf data.tar.gz --strip-components 1
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.uptickd/config/app.toml
sudo systemctl restart uptickd && sudo journalctl -u uptickd -f -o cat
```

# Create a service file
```bash
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
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable uptickd && \
sudo systemctl restart uptickd && sudo journalctl -u uptickd -f -o cat
```

## Create validator
```bash
uptickd tx staking create-validator \
--chain-id uptick_7000-1 \
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
```bash
sudo systemctl stop uptickd && \
sudo systemctl disable uptickd && \
rm /etc/systemd/system/uptickd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .uptickd && \
rm -rf uptick-v0.2.3 && \
rm -rf $(which uptickd)
```
