# Teritori testnet guide
![tert](https://user-images.githubusercontent.com/44331529/180614436-1041172a-0b1e-4df3-85b7-3d18899f3e43.png)

[EXPLORER 1](http://explorer.stavr.tech/Teritori/staking) \
[EXPLORER 2](https://explorer.nodestake.top/teritori-testnet/staking)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4| 8GB  | 100GB    |

# 1) Auto_install script
```python
wget -O teritor https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Teritori/Teritori%20testnet/teritor && chmod +x teritor && ./teritor
                
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## GO 19 (one command)
```python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

## Build 31.05.23
```python
git clone https://github.com/TERITORI/teritori-chain
cd teritori-chain
git checkout v1.4.0
make install
```
*******🟢UPDATE🟢******* 31.05.23
```python
cd $HOME/teritori-chain
git fetch --all
git checkout v1.4.0
make install
sudo systemctl restart teritorid && sudo journalctl -u teritorid -f -o cat
```
`teritorid version --long`
- version: v1.4.0
- commit: 01f60ec7fb9cb9d77dbe7fee1a9d69ff4fb0d2b9


```python
teritorid init STAVRguide --chain-id teritori-testnet-v3
teritorid config chain-id teritori-testnet-v3
```

## Create/recover wallet
```python
teritorid keys add <walletname>
    OR
teritorid keys add <walletname> --recover
```
### when creating, do not forget to write down the seed phrase

# Genesis
```python
cd $HOME
wget https://github.com/TERITORI/teritori-chain/raw/mainnet/testnet/teritori-testnet-v3/genesis.json
mv genesis.json .teritorid/config/
```

## Seeds,peers and gas price
```
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0utori\"/;" ~/.teritorid/config/app.toml
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.teritorid/config/config.toml

    peers="ccc59b8a55f9c6e7a24bd693e2796f781ea3a670@65.108.227.133:27656,5ae1012f9b0f4672d8152de903d115dd2f1a3ee3@65.21.170.3:27656,22101a61b235e607d5d0ad51b698d7511ebf87e2@65.108.43.227:26796,15dd94f68c450da2c3b7c60b6364e3dce6f0cbf2@185.193.66.68:26641"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.teritorid/config/config.toml
    seeds=""
    sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.teritorid/config/config.toml
```
## Download addrbook
```python
wget -O $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Teritori/Teritori%20testnet/addrbook.json"
```

## Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.teritorid/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.teritorid/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.teritorid/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.teritorid/config/app.toml

## Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.teritorid/config/config.toml

## StateSync
```python
peers="https://rpc-t.teritori.nodestake.top:443"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.teritorid/config/config.toml
SNAP_RPC=https://rpc-t.teritori.nodestake.top:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" ~/.teritorid/config/config.toml
teritorid tendermint unsafe-reset-all --home $HOME/.teritorid
sudo systemctl restart teritorid && journalctl -u teritorid -f -o cat
```
# Create a service file

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


    
# Start
```bash
sudo systemctl daemon-reload
sudo systemctl enable teritorid
sudo systemctl restart teritorid && sudo journalctl -u teritorid -f -o cat
```

[DISCORD FAUCET](https://discord.gg/zzJEmR8nhr)

## Create validator
    teritorid tx staking create-validator \
    --amount=1000000utori \
    --pubkey=$(teritorid tendermint show-validator) \
    --moniker="STAVRguide" \
    --identity="" \
    --details="" \
    --website="" \
    --chain-id="teritori-testnet-v3" \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.1" \
    --min-self-delegation="1" \
    --fees 500utori \
    --from=<walletname> -y

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Teritori)
=

# Delete node
```python
sudo systemctl stop teritorid
sudo systemctl disable teritorid
rm /etc/systemd/system/teritorid.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .teritorid
rm -rf teritori-chain
rm -rf $(which teritorid)
```


