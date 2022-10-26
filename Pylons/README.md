# Guide Pylons testnet
![pylo](https://user-images.githubusercontent.com/44331529/182419013-c3e5e07d-08de-4459-aa1c-88af51d6f340.png)

[Website](https://www.pylons.tech/home/)
=
[EXPLORER 1](https://pylons.explorers.guru/validators) \
[EXPLORER 2](https://wallet.pylons.tech/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   2| 4GB  | 100GB    |
### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y

## GO 18.3 (one command)
    ver="1.18.3" && \
    cd $HOME && \
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
    sudo rm -rf /usr/local/go && \
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
    rm "go$ver.linux-amd64.tar.gz" && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
    source $HOME/.bash_profile && \
    go version

# Binary   14.10.22
```console 
cd $HOME
git clone https://github.com/Pylons-tech/pylons
cd pylons
git checkout v1.0.0-rc5
make install
```
`pylonsd version version --long | head`
+ version: 1.0.0-rc5
+ commit: 9e173a70964d0d139d0288c2832cc027b98bd637

## Initialisation
```console
pylonsd init <moniker> --chain-id=pylons-testnet-3
```
## Add wallet
```console
pylonsd keys add <walletName>
pylonsd keys add <walletName> --recover
```
# Genesis
```console
wget -O $HOME/.pylons/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Pylons/genesis.json"

```

`sha256sum $HOME/.pylons/config/genesis.json`
- bdcd16fbb89ab8a329754377f174f32df077ea0956fe66751b1fc67a802da7cd

### Pruning (optional)
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.pylons/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.pylons/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.pylons/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.pylons/config/app.toml
```

### Indexer (optional)
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.pylons/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ubedrock\"/;" ~/.pylons/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.pylons/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.pylons/config/config.toml

peers="2c50b8171af784f1dca3d37d5dda5e90f1e1add8@95.214.55.4:26656,4f90babf520599ffe606157b0151c4c9bc0ec23f@194.163.172.115:26666,ebecc93e7865036fbdf8d3d54a624941d6e41ba1@104.200.136.57:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.pylons/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.pylons/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.pylons/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.pylons/config/config.toml
```

## Download addrbook
```console
wget -O $HOME/.pylons/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Pylons/addrbook.json"
```

# SnapShot 26.10.22 (4.7 GB) block height --> 2700740
```bash
# install the node as standard, but do not launch. Then we delete the .data directory and create an empty directory
sudo systemctl stop pylonsd
rm -rf $HOME/.pylons/data/
mkdir $HOME/.pylons/data/

# download archive
cd $HOME
wget http://pylons.snap.stavr.tech:7140/pylonsdata.tar.gz

# unpack the archive
tar -C $HOME/ -zxvf pylonsdata.tar.gz --strip-components 1
# !! IMPORTANT POINT. If the validator was created earlier. Need to reset priv_validator_state.json  !!
wget -O $HOME/.pylons/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/Canto/priv_validator_state.json"
cd && cat .pylons/data/priv_validator_state.json
{
  "height": "0",
  "round": 0,
  "step": 0
}
# after unpacking, run the node
# don't forget to delete the archive to save space
cd $HOME
rm pylonsdata.tar.gz
systemctl restart pylonsd && journalctl -u pylonsd -f -o cat
```


# Create a service file
```console
sudo tee /etc/systemd/system/pylonsd.service > /dev/null <<EOF
[Unit]
Description=Pylons
After=network-online.target

[Service]
User=$USER
ExecStart=$(which pylonsd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

# Start node (one command)
```console
sudo systemctl daemon-reload && \
sudo systemctl enable pylonsd && \
sudo systemctl restart pylonsd && \
sudo journalctl -u pylonsd -f -o cat
```

## Create validator
```
pylonsd tx staking create-validator \
--amount 1000000ubedrock \
--from <walletname> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(pylonsd tendermint show-validator) \
--moniker <Moniker> \
--chain-id pylons-testnet-3 \
--gas="auto" \
--fees 100udws
-y
```

### Delete node (one command)
```
sudo systemctl stop pylonsd && \
sudo systemctl disable pylonsd && \
rm /etc/systemd/system/pylonsd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .pylons && \
rm -rf pylons && \
rm -rf $(which pylonsd)
```
