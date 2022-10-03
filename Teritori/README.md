# Teritori Mainnet guide
![tert](https://user-images.githubusercontent.com/44331529/180614436-1041172a-0b1e-4df3-85b7-3d18899f3e43.png)

[EXPLORER](https://explorer.ericet.xyz/teritori/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  12| 32GB | 250GB    |

# 1) Auto_install script 
```bash
wget -O teritorm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Teritori/teritorm && chmod +x teritorm && ./teritorm
```
# 2) Manual installation

### Preparing the server
```bash
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 18.1 (one command)

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

## Build
```bash
git clone https://github.com/TERITORI/teritori-chain
cd teritori-chain 
git checkout v1.1.2
make install
```

```bash
teritorid init STAVRguide --chain-id teritori-1
teritorid config chain-id teritori-1
```

## Create/recover wallet
```bash
teritorid keys add <walletname>
teritorid keys add <walletname> --recover
```

### when creating, do not forget to write down the seed phrase

# Genesis
```bash
cd $HOME
wget -O ~/.teritorid/config/genesis.json https://media.githubusercontent.com/media/TERITORI/teritori-chain/v1.1.2/mainnet/teritori-1/genesis.json

```

## Seeds,peers and gas price
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0utori\"/;" ~/.teritorid/config/app.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.teritorid/config/config.toml

peers="3069b058b5ed85c3cdb2cf18fb1d255d966b53af@193.149.187.8:26656,a06fbbb9ace823ae28a696a91daa2d0644653c28@65.21.32.200:26756,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:15956"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.teritorid/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.teritorid/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.teritorid/config/addrbook.json "soon"
```

## Pruning (optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.teritorid/config/app.toml
```

## Indexer (optional)
```bash
ndexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.teritorid/config/config.toml
```
# Create a service file
```bash
sudo tee /etc/systemd/system/teritorid.service > /dev/null <<EOF
[Unit]
Description=Teritorid
After=network-online.target

[Service]
User=$USER
ExecStart=$(which teritorid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

    
# Start
```bash
sudo systemctl daemon-reload
sudo systemctl enable teritorid
sudo systemctl restart teritorid
sudo journalctl -u teritorid -f -o cat
```

## Create validator
```bash
teritorid tx staking create-validator \
--amount=1000000utori \
--pubkey=$(teritorid tendermint show-validator) \
--moniker="STAVRguide" \
--identity="" \
--details="" \
--website="" \
--chain-id="teritori-1" \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees 500utori \
--from=<walletname> -y
```

# Delete node
```bash
sudo systemctl stop teritorid && \
sudo systemctl disable teritorid && \
rm /etc/systemd/system/teritorid.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .teritorid && \
rm -rf teritori-chain && \
rm -rf $(which teritorid)
```


