# Provenance mainnet guide
![ppr (2)](https://user-images.githubusercontent.com/44331529/180606866-29524746-c733-43da-9af1-b6acf2a97eb3.png)
![ppr (1)](https://user-images.githubusercontent.com/44331529/180606868-07bdcdb1-ba8d-4b84-9cf0-23db6b13916c.png)


[Website](https://provenance.io/) \
[EXPLORER 1](https://explorer.stavr.tech/Provenance) \
[EXPLORER 2](https://www.mintscan.io/provenance/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 250GB    |

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip fail2ban libleveldb-dev -y
```

## GO 1.20 (one command)
```python
ver="1.20"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
# Build 07.07.23
```python
export PIO_HOME=~/.provenanced
git clone https://github.com/provenance-io/provenance.git && cd provenance
git checkout v1.16.0
make install
```

*******🟢UPDATE🟢******* 07.07.23
```python
export PIO_HOME=~/.provenanced
cd provenance
git fetch --all
git checkout v1.16.0
make build
mv $HOME/provenance/build/provenanced $(which provenanced)
provenanced version
#version: v1.16.0
#commit: 95064252
sudo systemctl restart provenanced && journalctl -u provenanced -f -o cat
```

`provenanced version`
- version: v1.16.0
- commit: 95064252

```python
provenanced init STAVRguide --chain-id pio-mainnet-1
```

## Create/recover wallet
```python
rovenanced keys add <walletname>
        OR
provenanced keys add <walletname> --recover
```
## Genesis
```python
wget -O $HOME/.provenanced/config/genesis.json "https://raw.githubusercontent.com/provenance-io/mainnet/main/pio-mainnet-1/genesis.json"
```
`sha256sum ~/.provenanced/config/genesis.json`
+ 8b10448662a55462fb23a8f4faedae2ec8a24aefe04d2e7475400c0367948940*


## Download addrbook
```python
wget -O $HOME/.provenanced/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Provenance/addrbook.json"
```

## Minimum gas price/Peers/Seeds
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"1905nhash\"/;" ~/.provenanced/config/app.toml
db_backend="cleveldb" && \
sed -i.bak -e "s/^db_backend *=.*/db_backend = \"$db_backend\"/" ~/.provenanced/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.provenanced/config/config.toml
peers="c4ffbe7e54790ee4b65e2152b2a6f65d15aeab4e@65.108.253.58:26657,286868295b6c56257332a8aca922f898353d2575@154.53.40.114:56651,de4e97e82e5fc567e55326383d46c72ae0ad7741@65.108.12.222:26757,358c97bb55717228f585491ef4c76d563183c583@194.163.165.174:26656,feb3bdc1c6f5ec32961c8051d9afec6984a59483@51.195.176.98:26658,666fca6c8f62f28fb4ab294589ce5d62b5823c91@161.97.115.247:26657"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.provenanced/config/config.toml
seeds="4bd2fb0ae5a123f1db325960836004f980ee09b4@seed-0.provenance.io:26656,048b991204d7aac7209229cbe457f622eed96e5d@seed-1.provenance.io:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.provenanced/config/config.toml
```
### Pruning (optional)
```python
runing="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.provenanced/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.provenanced/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.provenanced/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.provenanced/config/app.toml
```
### Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.provenanced/config/config.toml
```
# Create a service file
```python
sudo tee /etc/systemd/system/provenanced.service > /dev/null <<EOF
[Unit]
Description=provenanced
After=network-online.target

[Service]
User=$USER
Environment="PIO_HOME=$HOME/.provenanced"
ExecStart=$(which provenanced) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
## Start
```python
udo systemctl daemon-reload && \ 
sudo systemctl enable provenanced && \
sudo systemctl restart provenanced && \
sudo journalctl -u provenanced -f -o cat
```
## Create validator
```python
provenanced tx staking create-validator \
--amount 1000000nhash \
--from <walletName> \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.2" \
--commission-rate "0.05" \
--min-self-delegation "1" \
--details="" \
--identity="" \
--pubkey  $(provenanced tendermint show-validator) \
--moniker <moniker> \
--fees 555000000nhash \
--chain-id pio-mainnet-1 -y
```

## Delete node
```python
udo systemctl stop provenanced && \
sudo systemctl disable provenanced && \
rm /etc/systemd/system/provenanced.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .provenanced && \
rm -rf provenance && \
rm -rf $(which provenanced)
```

