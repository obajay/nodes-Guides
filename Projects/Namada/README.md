# Namada Testnet guide

![namada](https://github.com/obajay/nodes-Guides/assets/44331529/e1ae776c-13fb-40a5-9aaa-111d2d1fe84a)
# Namada is a LAYER 1 BLOCKCHAIN solution that redefines Asset-Agnostic, Multichain Privacy

[WebSite](https://namada.net/)\
[GitHub](https://github.com/anoma)
=


- 🔗Minimum hardware requirements:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Validator |   8| 16GB | 1TB      |

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

# Build 09.12.23
# 💻 Testnet 15
```python
cd $HOME
NAMADA_TAG="v0.28.0"
curl -L -o namada.tar.gz "https://github.com/anoma/namada/releases/download/$NAMADA_TAG/namada-${NAMADA_TAG}-Linux-x86_64.tar.gz"
tar -xvf namada.tar.gz
sudo mv namada-${NAMADA_TAG}-Linux-x86_64/* /usr/local/bin/
rm -rf namada-${NAMADA_TAG}-Linux-x86_64 namada.tar.gz
```

`namada --version`
- Namada v0.28.0

# 💡Preparing a validator for pre-genesis  
- [DOCS](https://docs.namada.net/operators/networks/genesis-flow/participants#generating-transactions)
```python
mkdir $HOME/.local/share/namada
ALIAS="your_moniker"
namadaw --pre-genesis key gen --alias $ALIAS
#enter and confirm password
🟢 backup this mnemonic phrase

TX_FILE_PATH="$HOME/.local/share/namada/pre-genesis/transactions.toml"
namadac utils init-genesis-established-account --path $TX_FILE_PATH --aliases $ALIAS
#enter and confirm password

ESTABLISHED_ACCOUNT_ADDRESS="your_established_account from previous step "
EMAIL="your_email"
SELF_BOND_AMOUNT=1000000
IP="your_ip:26656"
namadac utils init-genesis-validator --address $ESTABLISHED_ACCOUNT_ADDRESS --alias $ALIAS --net-address $IP --commission-rate 0.05 --max-commission-rate-change 0.01 --self-bond-amount $SELF_BOND_AMOUNT --email $EMAIL --path $TX_FILE_PATH

namadac utils sign-genesis-txs --path $TX_FILE_PATH --output $HOME/.local/share/namada/pre-genesis/signed-transactions.toml --alias $ALIAS
```

