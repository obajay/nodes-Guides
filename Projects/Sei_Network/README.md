# Guide install node - Sei_Network

![sei](https://user-images.githubusercontent.com/44331529/180607309-cc8df238-af95-451b-b99d-d858361aac51.png)

[EXPLORER 1](https://explorer.stavr.tech/sei/staking) \
[EXPLORER 2](https://sei.explorers.guru/validators) \
[EXPLORER 3](https://testnet-explorer.brocha.in/sei%20atlantic%202/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Atlantic-2|  16| 32GB | 500GB    |

# 1) Auto_install script
```python
wget -O sei4 https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Sei_Network/sei4 && chmod +x sei4 && ./sei4
```
# 2) Manual installation

## Server preparation
### Updating the repositories
```python
sudo apt update && sudo apt upgrade -y
```
### Installing the necessary utilities 
```python
      sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

### Build GO (one command)
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

# Installing the binaries (19.06.23)
```python
cd $HOME
git clone https://github.com/sei-protocol/sei-chain.git
cd sei-chain
git checkout 3.0.4
make install
```
`seid version --long | head`
+ version: 3.0.4
+ commit: dba16898f6db5a3289de3de9b05f044d7e0ae72d

    
## Initializing the node to create the necessary configuration files
```python
seid init STAVRguide --chain-id atlantic-2
seid config chain-id atlantic-2
```

## Downloading Genesis
```python
curl -Ls https://raw.githubusercontent.com/sei-protocol/testnet/main/atlantic-2/genesis.json > $HOME/.sei/config/genesis.json
```
### Let's check the genesis
`sha256sum ~/.sei/config/genesis.json`
+ 3c135db9177a428893353d7889149ca2ed9c075d6846be07af60354022b81318

## Set up node configuration
```python
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001usei\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^pruning *=.*|pruning = \"custom\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^pruning-keep-recent *=.*|pruning-keep-recent = \"3000\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^pruning-keep-every *=.*|pruning-keep-every = \"0\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^pruning-interval *=.*|pruning-interval = \"10\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^snapshot-interval *=.*|snapshot-interval = \"1000\"|" $HOME/.sei/config/app.toml
sed -i -e "s|^snapshot-keep-recent *=.*|snapshot-keep-recent = \"2\"|" $HOME/.sei/config/app.toml
```

## (OPTIONAL) Turn off indexing in config.toml
```pyton
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sei/config/config.toml
```
## Create a service file
```pyton
sudo tee /etc/systemd/system/seid.service > /dev/null <<EOF
[Unit]
Description=seid
After=network-online.target
    
[Service]
User=$USER
ExecStart=$(which seid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
## SnapShot by NodeJumper

[SnapShot](https://app.nodejumper.io/sei-testnet/sync)
=

## START
```python
sudo systemctl daemon-reload &&
sudo systemctl enable seid &&
sudo systemctl restart seid && sudo journalctl -u seid -f -o cat


## create wallet or restore wallet
```python
seid keys add <name_wallet>
     OR
seid keys add <name_wallet> --recover
```
# Don't forget to save the seed !!!

We take test coins in discord

## Create a validator
    seid tx staking create-validator \
    --chain-id atlantic-2\
    --commission-rate 0.05 \
    --commission-max-rate 0.2 \
    --commission-max-change-rate 0.1 \
    --min-self-delegation 1 \
    --amount 1000000usei \
    --pubkey $(seid tendermint show-validator) \
    --moniker "STAVRguide" \
    --from <name_wallet> \
    --fees 5550usei -y
    
## Delete Node
```python
sudo systemctl stop seid && \
sudo systemctl disable seid && \
rm /etc/systemd/system/seid.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .sei && \
rm -rf sei-chain && \
rm -rf $(which seid)
```
