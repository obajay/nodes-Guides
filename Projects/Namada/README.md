# Namada Testnet guide

![namada](https://github.com/obajay/nodes-Guides/assets/44331529/50d63960-23bc-4530-a103-04b2c13f48e4)
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
## 🟢backup this folder🟢
```python
$HOME/.local/share/namada/pre-genesis/
```

<h1 align="center"> 🚀 Next Step - We send all our work to github and wait for the merge</h1>


## ✨Follow the link to [Github](https://github.com/anoma/namada-testnets) and make a fork
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/ca89dee5-e38f-4dbb-839a-80f1ba727f02)
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/bb73eed0-87c3-419e-a8e0-9f1adcb92501)
## ✨In our fork we go to the `namada-public-testnet-15` repository
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/a2cd4536-ccfe-4b93-ab82-5cbcd5be205f)
## ✨then create a new file
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/15aed3c8-8f8d-493d-9981-de61626fd995)
## ✨we call it (we use our nickname, which we entered at the very beginning -> `ALIAS="your_moniker"`) and add the ending `.toml`
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/0550ec18-3a0e-4f18-9090-0a8e11acf8f8)
### ✨ Next on our server we go to the repository `$HOME/.local/share/namada/pre-genesis/`
find the `signed-transactions.toml` file. from there we copy all the data and paste it into our github
add `discord` `twitter` `telegram`. Let’s check again to see if everything is correct, and then click `Commit changes`
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/6ea4282a-18d8-4f29-aa9d-9759508d9930)
## ✨Since we are trying to take part in the testnet for the first time, it is important to specify `Create`
For those who are just updating their PR (they took part in previous testnets), they write `Update`
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/20970651-45f8-4043-9311-2df7e4454e65)
## ✨Next step -Pull Requests
![1212](https://github.com/obajay/nodes-Guides/assets/44331529/05fd6576-99ec-4b34-9ef8-1211b7de6ec4)
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/b90941b2-788a-4bdc-956e-c1fad110212d)
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/be430df3-0a29-43cd-a89a-172f74b30a05)
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/7a4e0cb3-7117-4792-9fc4-9ab3540b9c50)


<h1 align="center"> 🏆Congratulations. You have submitted your Pull Request🏆</h1>
