<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 25546872.1 | 0.00448278 | 114521.007642083 |

<!-- END_TABLE -->





[🔥OUR VALIDATOR🔥](https://restake.app/umee/umeevaloper1dkjcas3j43u3v6l94jhhhnjxhlnwxt3m02p4c3)
=

# Umee mainnet guide
![изображение](https://user-images.githubusercontent.com/44331529/180614842-60138156-dcfd-4dce-9ff2-c89fcc5c38dc.png)

[EXPLORER 1](https://explorer.stavr.tech/Umee/staking) \
[EXPLORER 2](https://www.mintscan.io/umee/validators)
=
- **Recommended hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB  | 500GB    |

# 1) Auto_install script
```python
wget -O Ume https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Umee/Ume && chmod +x Ume && ./Ume
```
# 2) Manual installation

### Preparing the server
```
sudo apt update
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop net-tools lsof -y < "/dev/null"
```
## Go
```
ver="1.21.3"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
# Binary   08.01.24
```python
cd $HOME
git clone https://github.com/umee-network/umee.git
cd umee
git checkout v6.3.0
make install
```
*******🟢UPDATE🟢******* 08.01.24
```python
cd $HOME/umee
git fetch --all
git checkout v6.3.0
make install
umeed version --long | grep -e commit -e version
#version: 6.3.0
#commit: cbf1d36e9deee4b086235a02e0fc9301e32bbcfc
sudo systemctl restart umeed && journalctl -u umeed -f -o cat
```

`umeed version --long`
+ version: 6.3.0
+ commit: cbf1d36e9deee4b086235a02e0fc9301e32bbcfc

## Init
```python
umeed init STAVRguide --chain-id umee-1
umeed config chain-id umee-1
```

## Create/recover wallet
```python
umeed keys add <walletname>
umeed keys add <walletname> --recover
```

## Genesis
```python
wget -O $HOME/.umee/config/genesis.json https://github.com/umee-network/mainnet/raw/main/genesis.json
```

`jq -S -c -M '' $HOME/.umee/config/genesis.json | shasum -a 256`
+ d4264b31472a922ec05620793dd8832a3a78032b23cdaa065f38ff3803f13800

    
### Peers/Gas

```python  
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uumee\"/;" ~/.umee/config/app.toml
peers="7f8b83fd029e33f5c69f2d3030b48e0785bd8af0@65.108.230.188:13656,5ab53dc31bf51e9416419112b418e4908519a97a@65.21.139.155:27656,1b728581c6d308078e2b969a0c6243852f77d28d@umee.peers.m.stavr.tech:10456"

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.umee/config/config.toml 
```
## Download addrbook
```python
wget -O $HOME/.umee/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Umee/addrbook.json"
```

## StateSync

```python
SNAP_RPC=https://umee.rpc.m.stavr.tech:443
peers="c014463cb2de618bef420e40f503c5e57decade4@umee.peers.m.stavr.tech:10456"
sed -i -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.umee/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 300)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.umee/config/config.toml
umeed tendermint unsafe-reset-all --home $HOME/.umee
wget -O $HOME/.umee/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Umee/addrbook.json"
sudo systemctl restart umeed && journalctl -u umeed -f -o cat
```
# SnapShot (~0.9 GB) updated every 5 hours
```python
cd $HOME
apt install lz4
sudo systemctl stop umeed
cp $HOME/.umee/data/priv_validator_state.json $HOME/.umee/priv_validator_state.json.backup
rm -rf $HOME/.umee/data
curl -o - -L http://umee.snapshot.stavr.tech:1000/umee/umee-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.umee --strip-components 2
wget -O $HOME/.umee/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Umee/addrbook.json"
mv $HOME/.umee/priv_validator_state.json.backup $HOME/.umee/data/priv_validator_state.json
sudo systemctl restart umeed && journalctl -u umeed -f -o cat
```


## Service file
```python
echo "[Unit]
Description=Umee Node
After=network.target 
    
[Service]
User=$USER
Type=simple
ExecStart=$(which umeed) start
Restart=always
RestartSec=3
LimitNOFILE=65535 
    
[Install]
WantedBy=multi-user.target" > $HOME/umeed.service
sudo mv $HOME/umeed.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
```  
## Start node
```python
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable umeed
sudo systemctl restart umeed && journalctl -u umeed -f -o cat
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Umee)
=

### Delete node
```python
sudo systemctl stop umeed && \
sudo systemctl disable umeed && \
rm /etc/systemd/system/umeed.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .umee && \
rm -rf umee && \
rm -rf $(which umeed)
```
