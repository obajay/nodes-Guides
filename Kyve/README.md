# Kyve testnet guide
![Kyve (2)](https://user-images.githubusercontent.com/44331529/180600823-b7f4a17d-c213-49b5-a1b9-cbe2e3b630e2.png)
![Kyve (1)](https://user-images.githubusercontent.com/44331529/180600827-c8beffd5-dcb3-4ded-a9d6-8f9aa6c0859f.png)


[EXPLORER 1](https://explorer.kyve.network/korellia/staking) \
[EXPLORER 2](https://kyve.explorers.guru/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| korellia  |   4| 8GB  | 160GB    |

### Preparing the server

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

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

# Build 19.07.22

    cd $HOME
    wget https://github.com/KYVENetwork/chain/releases/download/v0.6.3/chain_linux_amd64.tar.gz
    tar -xvzf chain_linux_amd64.tar.gz
    chmod +x chaind
    sudo mv chaind $HOME/go/bin/
    rm chain_linux_amd64.tar.gz
    cd
    
    chaind init <moniker> --chain-id korellia
    chaind config chain-id korellia

## Create/recover wallet

    chaind keys add <walletname>
    chaind keys add <walletname> --recover

## Genesis

    wget https://github.com/KYVENetwork/chain/releases/download/v0.0.1/genesis.json
    mv genesis.json ~/.kyve/config/genesis.json


## Peers/Seeds

    seeds="e56574f922ff41c68b80700266dfc9e01ecae383@18.156.198.41:26656"
    peers=""
    sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.kyve/config/config.toml

### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.kyve/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.kyve/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.kyve/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.kyve/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.kyve/config/config.toml

## State Sync

	SNAP_RPC="65.108.57.92:26657"
	peers="d040c94305f0b421df815d5375201b34f8cae999@65.108.57.92:26656"
	sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.kyve/config/config.toml

	LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
	BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
	TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

	echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

	sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
	s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
	s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
	s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
	s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.kyve/config/config.toml

	wget -qO $HOME/.kyve/config/addrbook.json https://api.testnet.run/addrbook-korellia.json
	chaind tendermint unsafe-reset-all --home $HOME/.kyve
	**sudo systemctl restart kyved && journalctl -u kyved -f -o cat**  (if node runnig)


# Create a service file

	sudo tee <<EOF > /dev/null /etc/systemd/system/kyved.service
	[Unit]
	Description=KYVE Chain-Node daemon
	After=network-online.target

	[Service]
	User=$USER
	ExecStart=$(which chaind) start
	Restart=on-failure
	RestartSec=10
	LimitNOFILE=infinity

	[Install]
	WantedBy=multi-user.target
	EOF

## Start

    sudo systemctl daemon-reload && \ 
    sudo systemctl enable kyved && \
    sudo systemctl restart kyved && \
    sudo journalctl -u kyved -f -o cat

## Create validator


	chaind tx staking create-validator \
	--amount 10000000000tkyve \
	--moniker="<moniker>" \
	--identity="<identity>" \
	--website="<website>" \
	--details="<any details>" \
	--commission-rate "0.10" \
	--commission-max-rate "0.20" \
	--commission-max-change-rate "0.05" \
	--min-self-delegation "1" \
	--pubkey "$(chaind tendermint show-validator)" \
	--from <your-key-name> \
	--chain-id korellia


## Delete node
    sudo systemctl stop aurad && \
    sudo systemctl disable aurad && \
    rm /etc/systemd/system/aurad.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .kyve && \
    rm -rf $(which chaind)


