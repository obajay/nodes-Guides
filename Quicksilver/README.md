### Updating the repositories  

    sudo apt update && sudo apt upgrade -y

### Installing the necessary utilities

    sudo apt install curl build-essential git wget jq make gcc tmux nvme-cli -y
    
### GO 1.18.1 (one command)

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

### Node installation
    git clone https://github.com/ingenuity-build/quicksilver.git --branch v0.4.0
    cd quicksilver
    make install
    quicksilverd version
#### version: v0.4.0
#### commit: 2b3d79696a06f3902105909849ce245ce917ddb7

### Initialize the node
    quicksilverd init <name_node> --chain-id killerqueen-1
    
### Create wallet or restore
    quicksilverd keys add <name_wallet>
            or
    quicksilverd keys add <name_wallet> --recover
### Download Genesis
    wget -O $HOME/.quicksilverd/config/genesis.json "https://raw.githubusercontent.com/ingenuity-build/testnets/main/killerqueen/genesis.json"
#### check genesis
    sha256sum ~/.quicksilverd/config/genesis.json
    3510dd3310e3a127507a513b3e9c8b24147f549bac013a5130df4b704f1bac75
### Setting the minimum price for gas
    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001uqck\"/;" ~/.quicksilverd/config/app.toml
### Setting up pruning with one command (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="50" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.quicksilverd/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.quicksilverd/config/app.toml

### Add seed/peers
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.quicksilverd/config/config.toml
    peers="e50848e299c7909245a9af690341ff27e21f7b69@185.214.134.154:46656,acde95e98c7b5023a4d2e2efb4f031927cc492b4@88.99.104.175:26656,6ac91620bc5338e6f679835cc604769a213d362f@139.59.56.24:36366,b281289df37c5180f9ff278be5e29964afa0c229@185.56.139.84:26656,98ef7d25e444ad39ec739c039a00baa7105a748b@128.199.115.134:11656,4f35ab6008fc46cc50b103a337ec2266400eca2e@148.251.50.79:26656,90f4459126152d21983f42c8e86bc899cd618af6@116.202.15.183:11656,abe7397ff92a4ca61033ceac127b5fc3a9a4217f@65.108.98.218:25095"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.quicksilverd/config/config.toml
    seeds="dd3460ec11f78b4a7c4336f22a356fe00805ab64@seed.killerqueen-1.quicksilver.zone:26656,8603d0778bfe0a8d2f8eaa860dcdc5eb85b55982@seed02.killerqueen-1.quicksilver.zone:27676"
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.quicksilverd/config/config.toml

### Disable indexing (optional)
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.quicksilverd/config/config.toml

(ВАЖНО) На этом этапе нужно воспользоваться снэпшотом

#### You will need to sync using state_sync. Synchronizing from scratch will not work because of the conflict of consensus.
    SNAP_RPC1="http://node02.quicktest-1.quicksilver.zone:26657" \
    && SNAP_RPC2="http://node03.quicktest-1.quicksilver.zone:26657"
    LATEST_HEIGHT=$(curl -s $SNAP_RPC2/block | jq -r .result.block.header.height) \
    && BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)) \
    && TRUST_HASH=$(curl -s "$SNAP_RPC2/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
    
    LATEST_HEIGHT=$(curl -s $SNAP_RPC2/block | jq -r .result.block.header.height) \
    && BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)) \
    && TRUST_HASH=$(curl -s "$SNAP_RPC2/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
    echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
    sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
    s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC1,$SNAP_RPC2\"| ; \
    s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
    s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.quicksilverd/config/config.toml
    quicksilverd unsafe-reset-all

### Create a service file
    sudo tee /etc/systemd/system/quicksilverd.service > /dev/null <<EOF
    [Unit]
    Description=quicksilver
    After=network-online.target
    
    [Service]
    User=$USER
    ExecStart=$(which quicksilverd) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535
    
    [Install]
    WantedBy=multi-user.target
    EOF
    
    sudo systemctl daemon-reload && \
    sudo systemctl enable quicksilverd && \
    sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat

#### After synchronization, we go to the discord and we request coins

### Create a validator
    quicksilverd tx staking create-validator \
    --chain-id killerqueen-1 \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.1 \
    --min-self-delegation="1" \
    --amount=1000000uqck \
    --pubkey $(quicksilverd tendermint show-validator) \
    --moniker "<name_moniker>" \
    --from=<name_wallet> \
    --gas="auto" \
    --fees 555uqck -y
    
