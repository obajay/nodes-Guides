# Cosmos Mainnet guide

![cosmos](https://user-images.githubusercontent.com/44331529/224945758-dc4f9c32-ea14-4573-b64b-3b8ef6c01616.png)

[WebSite](https://cosmos.network/)\
[GitHub](https://github.com/cosmos/gaia)
=
[EXPLORER 1](https://ping.pub/cosmos/staking) \
[EXPLORER 2](https://www.mintscan.io/cosmos/validators)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  16| 32GB | 500GB    |


# 1) Auto_install script
```python
wget -O cosmos https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cosmos/cosmos && chmod +x cosmos && ./cosmos
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19
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

# Build 15.03.23
```python
cd $HOME
git clone https://github.com/cosmos/gaia cosmos
cd gaia
git checkout v9.0.0
make install

```
*******🟢UPDATE🟢******* 15.03.23
```python
cd $HOME
wget https://github.com/cosmos/gaia/releases/download/v9.0.0/gaiad-v9.0.0-linux-amd64
chmod +x gaiad-v9.0.0-linux-amd64
mv gaiad-v9.0.0-linux-amd64 $HOME/go/bin/gaiad
sudo systemctl restart gaiad && journalctl -u gaiad -f -o cat
```

`gaiad version --long`
- version: v9.0.0
- commit: 682770f2410ab0d33ac7f0c7203519d7a99fa2b6

```python
gaiad init STAVRguide --chain-id cosmoshub-4
gaiad config chain-id cosmoshub-4
```    

## Create/recover wallet
```python
gaiad keys add <walletname>
  OR
gaiad keys add <walletname> --recover
```

## Download Genesis
```python
wget https://raw.githubusercontent.com/cosmos/mainnet/master/genesis/genesis.cosmoshub-4.json.gz
gzip -d genesis.cosmoshub-4.json.gz
mv genesis.cosmoshub-4.json ~/.gaia/config/genesis.json

```
`sha256sum $HOME/.gaia/config/genesis.json`
+ 6ad715c1ab5637e505e7248bb4366e79d5dec1a24f4fac7db33fead567041633

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0uatom\"/;" ~/.gaia/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.gaia/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.gaia/config/config.toml
peers="2db4db5e13338bbc9fe2af1faca8540e409e24f1@65.108.105.155:26656,4ddba29a7dfa740a4edeb5c620c963f67f951e1d@5.9.72.212:2000,7c6bdee5b71200405568dc25f4fd3d24c30d3b26@34.82.253.127:26656,e726816f42831689eab9378d5d577f1d06d25716@176.9.188.21:26656,b533749dfe0dc09eff1dfb2adf83108f9125ee1c@162.55.97.111:26656,a7e990dcb57a9ed3da058d9c7faa93584f628fd4@64.25.109.162:26656,267bf9112e4961dcee4f3bd861354ed750c55b91@65.109.19.176:26656,45174d354590991c7b32809281dc817cbb0f4e09@142.44.138.161:26656,59f70cf86eda6370ca948abb7b2f9db263e1fc3b@51.79.20.228:26656,803abd0b6b0478ab7f7e38dbda89902ca67f8778@65.21.90.137:11956,da22eea1473bf95ac1ea2a8f283d09159b3a8448@65.109.19.177:26656,992db7238f37c0197f5c57d44926cec289f1805e@23.88.18.142:26656,57b6404b031f6513bde381cfb8f3e96a6024e8ee@51.79.20.234:26656,72829b78b38408b03793ed389b9f16596b82c306@146.59.81.92:26656,05d870293f89e0698a8bd198e31f6ca17baa3a17@13.125.136.217:26656,3799720e77d9af4257abda457acba8b5fe70f4cf@146.59.253.118:23050,538348fa1eac998dad392a3f00f7b957042c3e84@15.235.53.86:11156,cf395b1ba2b8f9fe74fbd85b265b0e83c6a4771b@198.244.213.94:22256,dee3771d222681139d9df18d4e127d4f52820614@65.108.142.81:26656,1cf084b47a12d41cc73ce357c421c38e91c14760@65.108.201.154:2000,76c819c2f4a4302b265436fb4dc0369301ba8e3a@65.109.21.75:26656,3a94f1021e84bb54a640e5b1c1fe16827824e4f7@51.79.20.217:26656,b858ca4f3fed2c36b949cf67188b126e2542a39a@135.181.215.115:26726,f40a6e7d7168a3f2a5362cd37cbe6eac7a686056@185.229.119.178:26656,cf52e109b7015d5c21f50ab4331fb7062160ab6c@35.206.171.231:26656,76cb6275dcd71f43aecf3b8dddae08554b7cc6f5@51.79.20.226:26656,7fc4d44d4fdb02532f5338998d65973505edaaf2@51.79.20.235:26656,5d45bc48f6c0199c047e685fafb4309f53593f37@5.161.119.242:49656,6ea2ef7d3dd5d6967708a0b31eed85ba090a90a1@65.108.121.190:12010,76c65af180735315774497698ccf2091fb81284e@34.141.50.37:26656,5dec3f53ee44689531cf1e16d89f6b0fe559ead9@168.119.71.45:26656,347bd02478940696d81fbcb364b04360d5935612@185.232.68.12:26656,48fc4fe58d5392bda805212ba0c8e4e772dba1f9@142.132.158.93:14956,e829d4764a5cecc44b3414777853b34407b36601@185.16.39.179:26656,55c22f83bd9a10213b8b9610627ecd7d49c0a66a@135.181.62.108:26656,f5f8b96406a165d486be243723bfa7291db1cf62@35.230.170.155:26656,dd53fa5cfb6a604feb80860d47506d0dd84baa12@142.132.210.234:26656,84718db3de9588699b797965879d282061960293@51.79.20.219:26656,9edd51012df3a09395a48eb68a84723d6308e08c@35.212.116.100:26656,1279eae188599463661c3e2b9ab492615a6d7079@65.108.235.32:2010,36a282f4d7b0542e4fa691b303c4636909779c4e@95.216.240.248:26656,81062b9a8807a1229543b84bae2898c50a1b1dfc@52.211.169.132:26656,eb644d5ede024ce6083c0f1ca038eb41b257b795@3.210.252.30:26656,84569a6deb24db7e3e9fe1821ec07282d597d293@220.232.174.218:26656,137f98c8e22965e672744a3f8909c0f4c8cffc53@135.148.54.43:26656,c6f03336e99b15b104048a1af056063107389441@18.142.7.52:26656,dc3a1d1fbe0e477ebdb56fcee13ac6288fce2d87@65.108.141.62:26656,241b17dba97a2ed3c3747d12781fb86c9706e2d4@89.58.27.86:26656,9ef5a70d2218c4b599030eaad4f66e1da2ba5dda@35.73.70.202:26656,e0ab6c5cc86959853f499236b8297344802ac5f4@5.161.139.201:26656,c1e437f73b8889b78ea34981e7c349157ad80284@107.135.15.66:26656,4437ef919ce6f55a4c2672b9808cfb7e2393df37@45.79.135.21:26656,25d3ec5a00235fe95d7a87bab54f03b6ac1962ba@34.78.95.235:26656,89367eebb50a7da333d80dce71b5d5020eb01f84@23.90.70.43:49656,37dfe1ec33e9f88f378a61a32462d57d2baa5e74@65.108.99.140:26656,0ecd1883e4a0bfad8021a8caac404b33e820276c@148.251.121.154:26656,213857e741833d17275ea559bb2d0342398cec99@35.245.206.45:26656,2633bc088bcf96209b695734005952906b5c45e3@3.123.191.80:26656,a94dff85ed430f0475f41fe306c82b7eb7f6e858@51.91.153.78:31649,44390f449904199d22148d07822b1462048a38ef@74.118.139.150:26656,971ed177b284db42108187867cb8694df48ac742@95.217.205.41:26656,b9b99fbf40189c604ea618c4b99c61abc1489b70@18.140.125.215:26656,f6f5d71d0b9e29f2b86f47ce0d62b059b53009fc@74.118.143.238:26656,dff07399aeadf3f1b6edfac07f92a238112d3036@93.189.30.120:26656,4607d5a633db023f4618149ea443aec361e904db@18.140.170.32:26656,4ced94cd9bb0b8c314559f878c4dff16ca3cf24b@138.201.63.42:26656,3da88430414ec9084c8983fe4d462cce655ff1f3@51.222.245.114:26656,4e18c2a64f190a4bc3afb57e96b32c02ee08d355@95.216.98.181:26656,c5bf14906ba28dcb389e055f824dabe9576ed3f4@52.87.182.81:26656,67685d93f2256caa7a2d53e3a104f9e437c3d247@95.216.114.244:26656,7dd34d8d3880bc48eff3e47b941d06bd1941a962@93.115.25.106:26656,61afb0f37c02031f285f6b27ead2a3e7a97cc28a@35.212.34.104:26656,9d048653fa4d98e6c0760ed0c54ad2d257ba46df@65.108.137.34:26656,32bdba6ced12cdf2e534566e6c3d66ee2f7ef494@84.244.95.229:26656,c14d39422b5d70d9084d19d286c7427c0762cdfc@162.55.92.114:2010,0eae0c3b87453c625a1de230fca4993b8ebe5c00@65.21.94.45:26656,5b143d463427d9ad0b621f97c0b8933643e293da@35.212.90.144:26656,89757803f40da51678451735445ad40d5b15e059@169.155.168.135:26656,1cce99042f884d669e7287e3e362bff8e385c63e@46.4.79.183:26726,e4e13104fb1266aa601e9610ee39b469436ea26a@157.90.237.105:26615,ea1779f3c46730e98727fbc0499ba45b31a40ce0@95.216.16.205:14956,71950462041283273efa597db443c556e70a9c17@3.38.173.31:26656,64148c47e1424173e3dcf90ab90bf196c2971b15@88.218.224.118:26656,fe21dd474640247888fc7c4dce82da8da08a8bfd@135.181.113.227:26656,f9fd30519c915ef1aeb63e99e345f83f08ec69d9@3.208.33.221:26656,d82513849c2722a32e2e03617a516f65b3ca0216@176.191.97.120:33656,8698cb819c9a4503fe2c71055f1380d08edc5adf@204.16.244.116:26656,4ebf074e8b4a24438bd0bd503b62b4728dfb8eae@35.212.101.35:26656,d53ecee926a66a4a6b1858004f5f22f77faca036@3.69.52.20:26656,0b27d23a50a6969a22b268ec90a198c31b741b2d@65.108.103.184:27656,b0ac7f1485eedfc063af251fe12d93a68a22131d@65.108.137.38:26656,ca5011c44fd74d95e7fca487c69e301df195750c@65.108.122.246:26726,1da54d20c7339713f1d6d28dd2117087dd33d0ca@5.9.59.145:26656,7b15dce221b13ca353187b4f7219a94db6b71ad3@185.119.118.109:2000,344d87e04fdf04be760da5069a59d9a489b886a6@52.14.44.1:26656,27ad834c62dbefc5beb74be7575515927bd07c58@193.176.85.151:26656,dea13e7232642331360d4387b0ab106b014092d4@116.202.236.59:26656,6b4a6e35c5329b07f23662c683e24b262dd10b5e@195.189.96.115:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.gaia/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.gaia/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.gaia/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.gaia/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gaia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gaia/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.gaia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.gaia/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.gaia/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.gaia/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Cosmos/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/gaiad.service > /dev/null <<EOF
[Unit]
Description=gaia
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gaiad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Gaia Mainnet
```python
SOON
```
# SnapShot Testnet (~0.2GB) updated every 5 hours  
```python
SOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable gaiad
sudo systemctl restart gaiad && sudo journalctl -u gaiad -f -o cat
```

### Create validator
```python
gaiad tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 1000000uatom \
--pubkey $(gaiad tendermint show-validator) \
--from <wallet> \
--moniker="STAVRguide" \
--chain-id cosmoshub-4 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```

## Delete node
```python
sudo systemctl stop gaiad && \
sudo systemctl disable gaiad && \
rm /etc/systemd/system/gaiad.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf gaia && \
rm -rf .gaia && \
rm -rf $(which gaiad)
```
#
### Sync Info
```python
gaiad status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
gaiad status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u gaiad -f -o cat
```
### Check Balance
```python
gaiad query bank balances cosmos...addressjkl1yjgn7z09ua9vms259j
```
