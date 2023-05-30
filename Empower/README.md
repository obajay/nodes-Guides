# Empower Testnet Guide

![empower](https://user-images.githubusercontent.com/44331529/193969092-38e7ec7f-bca0-4bd9-a31d-5ba52b71ec81.png)

[Website](https://www.empowerchain.io/)
=
[EXPLORER 1](http://explorer.stavr.tech/empower/staking) \
[EXPLORER 2](https://testnet.ping.pub/empower)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 160GB    |

# 1) Auto_install script
```python
wget -O empw https://raw.githubusercontent.com/obajay/nodes-Guides/main/Empower/empw && chmod +x empw && ./empw
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 18.3 (one command)
```python
cd $HOME && version="1.18.3" && \
wget "https://golang.org/dl/go$version.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$version.linux-amd64.tar.gz" && \
rm "go$version.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```


# Binary   25.05.23
```python
git clone https://github.com/EmpowerPlastic/empowerchain &&
cd empowerchain &&
git checkout circulus-1 &&
cd chain &&
make install

```
*******🟢UPDATE🟢******* 00.00.23

```python
SOOON
```

`empowerd version --long | head`
+ 0.0.3-747-g6cc06dd
+ commit: 6cc06dd568fa531685cff9b27d1256d072e4e0da


## Initialisation
```python
empowerd init STAVRguide --chain-id circulus-1 && \
empowerd config chain-id circulus-1

```
## Add wallet
```python
empowerd keys add <walletName>
empowerd keys add <walletName> --recover
```
# Genesis
```python
wget -O $HOME/.empowerchain/config/genesis.json "https://raw.githubusercontent.com/EmpowerPlastic/empowerchain/main/testnets/circulus-1/genesis.json"
```

`sha256sum $HOME/.empowerchain/config/genesis.json`
- ee87e0e4cb31cb53e75be8c89a11897be11a395b271b384b030a92f9873e1ccf

### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.empowerchain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.empowerchain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.empowerchain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.empowerchain/config/app.toml
```

### Indexer (optional)
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.empowerchain/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0umpwr\"/" $HOME/.empowerchain/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.empowerchain/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.empowerchain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.empowerchain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.empowerchain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.empowerchain/config/config.toml
```

## Download addrbook
```python
##SOOON   wget -O $HOME/.empowerchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Empower/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/empowerd.service > /dev/null <<EOF
[Unit]
Description=EmpowerChain Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which empowerd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

# Start node (one command)
```python
sudo systemctl daemon-reload && sudo systemctl enable empowerd
sudo systemctl restart empowerd && sudo journalctl -u empowerd -f -o cat
```

## Create validator
```python
empowerd tx staking create-validator \
--amount 1000000umpwr \
--from <walletname> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(empowerd tendermint show-validator) \
--moniker STAVRguide \
--chain-id circulus-1 \
-y
```

### Delete node (one command)
```python
sudo systemctl stop empowerd && \
sudo systemctl disable empowerd && \
rm /etc/systemd/system/empowerd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .empowerchain && \
rm -rf empowerchain && \
rm -rf $(which empowerd)
```
