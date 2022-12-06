# Aura testnet guide

![Aura](https://user-images.githubusercontent.com/44331529/180595364-72b306db-c60b-463e-877c-57ee5acc126e.png)
![aaura](https://user-images.githubusercontent.com/44331529/180595514-1dfc72a9-b72e-477b-ab5b-54f8a5071c7d.png)



[EXPLORER](https://testnet.owlstake.com/Aura-Network/staking)

- **Minimum hardware requirements**:

| Node Type  | RAM  | Storage  | 
|------------|------|----------|
| Validator  | 16GB | 500GB    |

# 1) Auto_install script
```python
wget -O aur https://raw.githubusercontent.com/obajay/nodes-Guides/main/Aura/aur && chmod +x aur && ./aur
```

# 2) Manual instruction
### Preparing the server

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## GO 18.1 (one command)

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

# Build 25.11.22
```python
cd $HOME
git clone https://github.com/aura-nw/aura && cd aura
git checkout euphoria_v0.4.1
make install
```

*******🟢UPDATE🟢******* 07.12.22

```python
cd $HOME/aura
git fetch --all
git checkout euphoria_v0.4.2
make install
sudo systemctl restart aurad && journalctl -u aurad -f -o cat
```

`aurad version --long | head`
+ version: euphoria_v0.4.2
+ commit: 7922c61c8c404645007995211070c9d817713c33

```python
aurad init STAVRguide --chain-id euphoria-2
```

## Create/recover wallet

    aurad keys add <walletname>
    aurad keys add <walletname> --recover

## Genesis
```python
wget https://github.com/aura-nw/testnets/raw/main/euphoria-2/euphoria-2-genesis.tar.gz
tar -xzvf euphoria-2-genesis.tar.gz
mv euphoria-2-genesis.json $HOME/.aura/config/genesis.json
```
`sha256sum ~/.aura/config/genesis.json`
+ b0ee9ed933ac5c24697637bc56335136211e8d26962b1f8622c626c90772b0d6

## Download addrbook
```python
wget -O $HOME/.aura/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Aura/addrbook.json"
```

## Minimum gas price/Peers/Seeds
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ueaura\"/;" ~/.aura/config/app.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.aura/config/config.toml
peers="7cad1bcb2ad777dba21840832341f2ce14bae1a5@5.75.174.126:26656,705e3c2b2b554586976ed88bb27f68e4c4176a33@13.250.223.114:26656,b9243524f659f2ff56691a4b2919c3060b2bb824@13.214.5.1:26656,d334e2b9dd84346ea532ff3d43f3f7c4946845c9@144.91.122.166:26656,b91ee5c72905bc49beed2720bb882c923c68fbc9@65.108.142.47:21656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.aura/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.aura/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.aura/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.aura/config/config.toml
```

### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.aura/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.aura/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.aura/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.aura/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.aura/config/config.toml

## State Sync
```python
sudo systemctl stop aurad
cp $HOME/.aura/data/priv_validator_state.json $HOME/.aura/priv_validator_state.json.backup
aurad tendermint unsafe-reset-all --home $HOME/.aura
STATE_SYNC_RPC=https://aura-testnet.rpc.kjnodes.com:443
STATE_SYNC_PEER=d5519e378247dfb61dfe90652d1fe3e2b3005a5b@aura-testnet.rpc.kjnodes.com:17656
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 2000))
SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -e "s|^enable *=.*|enable = true|" $HOME/.aura/config/config.toml
sed -i.bak -e "s|^rpc_servers *=.*|rpc_servers = \"$STATE_SYNC_RPC,$STATE_SYNC_RPC\"|" \
  $HOME/.aura/config/config.toml
sed -i.bak -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" \
  $HOME/.aura/config/config.toml
sed -i.bak -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" \
  $HOME/.aura/config/config.toml
sed -i.bak -e "s|^persistent_peers *=.*|persistent_peers = \"$STATE_SYNC_PEER\"|" \
  $HOME/.aura/config/config.toml
mv $HOME/.aura/priv_validator_state.json.backup $HOME/.aura/data/priv_validator_state.json
curl -L https://snapshots.kjnodes.com/aura-testnet/wasm_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.aura
sudo systemctl restart aurad && sudo journalctl -u aurad -f -o cat

```
# SnapShot (~0.4 GB) updated every 5 hours
```python
cd $HOME
sudo systemctl stop aurad
cp $HOME/.aura/data/priv_validator_state.json $HOME/.aura/priv_validator_state.json.backup
rm -rf $HOME/.aura/data
curl -o - -L http://aura.snapshot.stavr.tech:5015/aura/aura-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.aura --strip-components 2
curl -o - -L http://aura.wasm.stavr.tech:1001/wasm-aura.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.aura --strip-components 2
mv $HOME/.aura/priv_validator_state.json.backup $HOME/.aura/data/priv_validator_state.json
sudo systemctl restart aurad && journalctl -u aurad -f -o cat
```

# Create a service file

    sudo tee /etc/systemd/system/aurad.service > /dev/null <<EOF
    [Unit]
    Description=aurad
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which aurad) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

## Start

    sudo systemctl daemon-reload && \
    sudo systemctl enable aurad && \
    sudo systemctl restart aurad && sudo journalctl -u aurad -f -o cat


## Create validator


    aurad tx staking create-validator \
    --amount 1000000ueaura \
    --from <walletName> \
    --commission-max-change-rate "0.1" \
    --commission-max-rate "0.2" \
    --commission-rate "0.05" \
    --min-self-delegation "1" \
    --details="" \
    --identity="" \
    --pubkey  $(aurad tendermint show-validator) \
    --moniker STAVRguide \
    --fees 555ueaura \
    --chain-id euphoria-2 -y


## Delete node
    sudo systemctl stop aurad && \
    sudo systemctl disable aurad && \
    rm /etc/systemd/system/aurad.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .aura && \
    rm -rf aura && \
    rm -rf $(which aurad)

