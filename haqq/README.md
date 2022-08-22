# Guide haqq
![haqq](https://user-images.githubusercontent.com/44331529/185350224-62b92bc1-bd4e-4ce7-a56b-0abfc631c95c.png)

[Website](https://islamiccoin.net/)
=
[EXPLORER](https://explorer.nodestake.top/haqq-testnet/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4| 8GB  | 100GB    |

# 1) Auto_install script
```bash 
wget -O haqq1 https://raw.githubusercontent.com/obajay/nodes-Guides/main/haqq/haqq1 && chmod +x haqq1 && ./haqq1
```
# 2) Manual installation

### Preparing the server

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y

## GO 18.3 (one command) 
```
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

# Binary   11.07.22
```console 
cd $HOME && \
git clone -b v1.0.3 https://github.com/haqq-network/haqq && \
cd haqq && \
make install


```
`haqqd -v`
- version: 1.0.3
- commit: 58215364d5be4c9ab2b17b2a80cf89f10f6de38a 

## Initialisation
```console
haqqd init <moniker-name> --chain-id=haqq_53211-1
```
## Add wallet
```console
haqqd keys add <walletName>
haqqd keys add <walletName> --recover
```
# Genesis
```console
curl -OL https://storage.googleapis.com/haqq-testedge-snapshots/genesis.json
mv genesis.json $HOME/.haqqd/config/genesis.json
```

`sha256sum $HOME/.haqqd/config/genesis.json`
- 9a7746dd6d4cc1e29f8a67759d90ef37af8bc78a7be3dd94538e94f42bc1f646  genesis.json

### Pruning (optional) one command
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.haqqd/config/app.toml
```
### Indexer (optional) one command
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.haqqd/config/config.toml

### Set up the minimum gas price and Peers/Seeds/Filter peers
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0stake\"/;" ~/.haqqd/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.haqqd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.haqqd/config/config.toml

peers="e15e1867f68011f80f2763e6691c89c923bf2f24@78.46.16.236:41656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.haqqd/config/config.toml

seeds="74d8f92c37ffe4c6393b3718ca531da8f0bf0594@seed1.haqqd.services:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.haqqd/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.haqqd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.haqqd/config/config.toml
```

## Download addrbook
```console
wget -O $HOME/.haqqd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/haqq/addrbook.json"
```

# Create a service file
```console
sudo tee /etc/systemd/system/haqqd.service > /dev/null <<EOF
[Unit]
Description=haqqd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which haqqd) start
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
sudo systemctl enable haqqd && \
sudo systemctl restart haqqd && \
sudo journalctl -u haqqd -f -o cat
```

## Create validator
```
haqqd tx staking create-validator \
--amount 1000000000000000000aISLM \
--from <walletName> \
--commission-max-change-rate "0.10" \
--commission-max-rate "0.20" \
--commission-rate "0.10" \
--min-self-delegation "1" \
--identity="" \
--details="" \
--website="" \
--pubkey $(haqqd tendermint show-validator) \
--moniker <Moniker> \
--chain-id haqq_53211-1 \
-y
```

### Delete node (one command)
```
sudo systemctl stop haqqd && \
sudo systemctl disable haqqd && \
rm /etc/systemd/system/haqqd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .haqqd && \
rm -rf haqq && \
rm -rf $(which haqqd)
```

