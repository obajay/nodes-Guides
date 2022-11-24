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
```bash
wget -O empw https://raw.githubusercontent.com/obajay/nodes-Guides/main/Empower/empw && chmod +x empw && ./empw
```
# 2) Manual installation

### Preparing the server
```bash
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 18.3 (one command)
```bash
cd $HOME && version="1.18.3" && \
wget "https://golang.org/dl/go$version.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$version.linux-amd64.tar.gz" && \
rm "go$version.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Binary   24.11.22
```bash
cd $HOME && git clone https://github.com/empowerchain/empowerchain && \
cd empowerchain/chain
git checkout v0.0.3
make build
mv $HOME/empowerchain/chain/build/empowerd $HOME/go/bin/
```
`empowerd version --long | head`
+ 0.0.3
+ commit: 9dc9ccedc17233bfd8ef49f248568014f72f7acb


## Initialisation
```bash
empowerd init STABRguide --chain-id altruistic-1 && \
empowerd config chain-id altruistic-1

```
## Add wallet
```console
empowerd keys add <walletName>
empowerd keys add <walletName> --recover
```
# Genesis
```bash
rm -rf $HOME/.empowerchain/config/genesis.json && cd $HOME/.empowerchain/config && wget https://raw.githubusercontent.com/empowerchain/empowerchain/main/testnets/altruistic-1/genesis.json
```

`sha256sum $HOME/.empowerchain/config/genesis.json`
- fcae4a283488be14181fdc55f46705d9e11a32f8e3e8e25da5374914915d5ca8

### Pruning (optional)
```bash
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
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.empowerchain/config/config.toml
```
### Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025umpwr\"/" $HOME/.empowerchain/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.empowerchain/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.empowerchain/config/config.toml

peers="9de92b545638f6baaa7d6d5109a1f7148f093db3@65.108.77.106:26656,4fd5e497563b2e09cfe6f857fb35bdae76c12582@65.108.206.56:26656,fe32c17373fbaa36d9fd86bc1146bfa125bb4f58@5.9.147.185:26656,220fb60b083bc4d443ce2a7a5363f4813dd4aef4@116.202.236.115:26656,225ad85c594d03942a026b90f4dab43f90230ea0@88.99.3.158:26656,2a2932e780a681ddf980594f7eacf5a33081edaf@192.168.147.43:26656,333de3fc2eba7eead24e0c5f53d665662b2ba001@10.132.0.11:26656,4a38efbae54fd1357329bd583186a68ccd6d85f9@94.130.212.252:26656,52450b21f346a4cf76334374c9d8012b2867b842@167.172.246.201:26656,56d05d4ae0e1440ad7c68e52cc841c424d59badd@192.168.1.46:26656,6a675d4f66bfe049321c3861bcfd19bd09fefbde@195.3.223.204:26656,1069820cdd9f5332503166b60dc686703b2dccc5@138.201.141.76:26656,277ff448eec6ec7fa665f68bdb1c9cb1a52ff597@159.69.110.238:26656,3335c9458105cf65546db0fb51b66f751eeb4906@5.189.129.30:26656,bfb56f4cb8361c49a2ac107251f92c0ea5a1c251@192.168.1.177:26656,edc9aa0bbf1fcd7433fcc3650e3f50ab0becc0b5@65.21.170.3:26656,d582bcd8a8f0a20c551098571727726bc75bae74@213.239.217.52:26656,eb182533a12d75fbae1ec32ef1f8fc6b6dd06601@65.109.28.219:26656,b22f0708c6f393bf79acc0a6ca23643fe7d58391@65.21.91.50:26656,e8f6d75ab37bf4f08c018f306416df1e138fd21c@95.217.135.41:26656,ed83872f2781b2bdb282fc2fd790527bcb6ffe9f@192.168.3.17:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.empowerchain/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.empowerchain/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.empowerchain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.empowerchain/config/config.toml
```

## Download addrbook
```bash
wget -O $HOME/.empowerchain/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Empower/addrbook.json"
```

# Create a service file
```console
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
```console
sudo systemctl daemon-reload && sudo systemctl enable empowerd
sudo systemctl restart empowerd && sudo journalctl -u empowerd -f -o cat
```

## Create validator
```
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
--chain-id altruistic-1 \
-y
```

### Delete node (one command)
```
sudo systemctl stop empowerd && \
sudo systemctl disable empowerd && \
rm /etc/systemd/system/empowerd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .empowerchain && \
rm -rf empowerchain && \
rm -rf $(which empowerd)
```
