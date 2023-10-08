# Realio testnet guide

![realio](https://user-images.githubusercontent.com/44331529/194716535-b96b71ad-b191-4c28-a112-1e71e0859e0a.png)

[Website](https://www.realio.fund/)
=
[EXPLORER 1](https://explorer.stavr.tech/Realio/staking) \
[EXPLORER 2](https://test.anode.team/realio/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4| 8GB | 160GB    |


# 1) Auto_install script
```python
soon
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 18.5

```python
cd $HOME
ver="1.18.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 03.03.23
```python
cd $HOME
git clone https://github.com/realiotech/realio-network.git
cd realio-network
git checkout v0.8.0-rc4
make install
```
`realio-networkd version --long`
- version: 0.8.0-rc4
- commit: 692d8ccbd4c229135445d82b51bd2dfd52224651

```python
realio-networkd init STAVRguide --chain-id realionetwork_3300-2
realio-networkd config chain-id realionetwork_3300-2
```    

## Create/recover wallet
```python
realio-networkd keys add <walletname>
realio-networkd keys add <walletname> --recover
```

## Download Genesis

```python
wget -O $HOME/.realio-network/config/genesis.json https://raw.githubusercontent.com/realiotech/testnets/main/realionetwork_3300-2/genesis.json
```
`sha256sum $HOME/.realio-network/config/genesis.json`
+ 7e3fef8375567d9cbffbbc9e32907c3c87143fd79ef2b6a1f13d232d89057ddc

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ario\"/" $HOME/.realio-network/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.realio-network/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.realio-network/config/config.toml
peers="ec2dbd6e5d25501c50fb8585b5678a7460ef11da@144.126.196.99:26656,5bd91f6e7e3bcaaddead32fd37d67458723fec73@159.223.132.183:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.realio-network/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.realio-network/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.realio-network/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.realio-network/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.realio-network/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.realio-network/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.realio-network/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.realio-network/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.realio-network/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.realio-network/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Realio/Testnet/addrbook.json"
```
# Create a service file
```python
sudo tee /etc/systemd/system/realio-networkd.service > /dev/null <<EOF
[Unit]
Description=realio
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which realio-networkd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload && sudo systemctl enable realio-networkd
sudo systemctl restart realio-networkd && sudo journalctl -u realio-networkd -f -o cat
```

### Create validator
```python
realio-networkd tx staking create-validator \
  --amount=1000000000000000000ario \
  --pubkey=$(realio-networkd tendermint show-validator) \
  --moniker="STAVRguide" \
  --chain-id=realionetwork_3300-2 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.1" \
  --min-self-delegation="1" \
  --fees 5000000000000000ario \
  --gas 800000
  --from=<walletName>
```

## Delete node
```python
sudo systemctl stop realio-networkd && \
sudo systemctl disable realio-networkd && \
rm /etc/systemd/system/realio-networkd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf realio-network && \
rm -rf .realio-network && \
rm -rf $(which realio-networkd)
```

