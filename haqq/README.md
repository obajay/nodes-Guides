# Guide haqq
![haqq](https://user-images.githubusercontent.com/44331529/185350224-62b92bc1-bd4e-4ce7-a56b-0abfc631c95c.png)

[Website](https://islamiccoin.net/)
=
[EXPLORER 1](https://testnet.manticore.team/haqq/staking) \
[EXPLORER 2](https://explorer.nodestake.top/haqq-testedge/staking)
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
```bash
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
```

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

# Binary   04.10.22
```bash 
git clone https://github.com/haqq-network/haqq
cd haqq
git checkout v1.2.0
make install
```

## Initialisation
```bash
haqqd init STAVRguide --chain-id=haqq_54211-3
```
## Add wallet
```bash
haqqd keys add <walletName>
haqqd keys add <walletName> --recover
```
# Genesis
```bash
cd $HOME/.haqqd/config && rm -rf genesis.json && wget https://github.com/haqq-network/validators-contest/raw/master/genesis.json
```

`sha256sum $HOME/.haqqd/config/genesis.json`
- b93f2650bdf560cab2cf7706ecee72bfba6d947fa57f8b1b8cb887f8b428233f  genesis.json

### Pruning
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
### Indexer (optional) one command
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.haqqd/config/config.toml

### Set up the minimum gas price and Peers/Seeds/Filter peers
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0stake\"/;" ~/.haqqd/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.haqqd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.haqqd/config/config.toml

seeds="62bf004201a90ce00df6f69390378c3d90f6dd7e@seed2.testedge2.haqq.network:26656,23a1176c9911eac442d6d1bf15f92eeabb3981d5@seed1.testedge2.haqq.network:26656"
peers="b3ce1618585a9012c42e9a78bf4a5c1b4bad1123@65.21.170.3:33656,952b9d918037bc8f6d52756c111d0a30a456b3fe@213.239.217.52:29656,85301989752fe0ca934854aecc6379c1ccddf937@65.109.49.111:26556,d648d598c34e0e58ec759aa399fe4534021e8401@109.205.180.81:29956,f2c77f2169b753f93078de2b6b86bfa1ec4a6282@141.95.124.150:20116,eaa6d38517bbc32bdc487e894b6be9477fb9298f@78.107.234.44:45656,37513faac5f48bd043a1be122096c1ea1c973854@65.108.52.192:36656,d2764c55607aa9e8d4cee6e763d3d14e73b83168@66.94.119.47:26656,fc4311f0109d5aed5fcb8656fb6eab29c15d1cf6@65.109.53.53:26656,297bf784ea674e05d36af48e3a951de966f9aa40@65.109.34.133:36656,bc8c24e9d231faf55d4c6c8992a8b187cdd5c214@65.109.17.86:32656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.haqqd/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.haqqd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.haqqd/config/config.toml
```

# SnapShot 21.10.22 (0.1 GB) block height --> 594004
```bash
# install the node as standard, but do not launch. Then we delete the .data directory and create an empty directory
sudo systemctl stop haqqd
rm -rf $HOME/.haqqd/data/
mkdir $HOME/.haqqd/data/

# download archive
cd $HOME
wget http://haqq.snapshot.stavr.tech:7150/haqqddata.tar.gz

# unpack the archive
tar -C $HOME/ -zxvf haqqddata.tar.gz --strip-components 1
# !! IMPORTANT POINT. If the validator was created earlier. Need to reset priv_validator_state.json  !!
wget -O $HOME/.haqqd/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/Canto/priv_validator_state.json"
cd && cat .haqqd/data/priv_validator_state.json

# after unpacking, run the node
# don't forget to delete the archive to save space
cd $HOME
rm haqqddata.tar.gz
systemctl restart haqqd && journalctl -u haqqd -f -o cat
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
--moniker STAVRguide \
--chain-id haqq_54211-3 \
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
#
### Sync Info
```bash
haqqd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```bash
haqqd status 2>&1 | jq .NodeInfo
```
### Check node logs
```bash
sudo journalctl -u haqqd -f -o cat
```
### Check Balance
```bash
haqqd query bank balances haqq...addresshaqq1yjgn7z09ua9vms259j
```
