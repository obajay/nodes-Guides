# Rizon mainnet guide

![rizon (2)](https://user-images.githubusercontent.com/44331529/180607172-a7f5cf1f-d1e7-4e9c-b3a0-a51871c2992d.png)
![rizon (1)](https://user-images.githubusercontent.com/44331529/180607173-f918aff4-499b-4996-bc2c-3f1fd5bc3d6f.png)

[Website](https://rizon.world/) \
[EXPLORER 1](https://explorer.stavr.tech/rizon/staking) \
[EXPLORER 2](https://www.mintscan.io/rizon/validators)
=
- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 200GB    |

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

# Build 15.10.22
```bash
git clone https://github.com/rizon-world/rizon
cd rizon
git checkout v0.4.1
make install
```

```bash
rizond init STAVRguide --chain-id titan-1
rizond config chain-id titan-1
```

## Create/recover wallet
```bash
rizond keys add <walletname>
rizond keys add <walletname> --recover
```
## Genesis
```java
wget -O $HOME/.rizon/config/genesis.json "https://raw.githubusercontent.com/rizon-world/mainnet/master/genesis.json"
```
`sha256sum ~/.rizon/config/genesis.json`
+ 6d5602e3746affea1096c729768bffd1f60633dfe88ae705f018d70fd3e90302

## Download addrbook (if need)

    wget -O $HOME/.rizon/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Rizon/addrbook.json"


## Minimum gas price/Peers/Seeds

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0stake\"/;" ~/.rizon/config/app.toml
    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.rizon/config/config.toml
    peers="0d51e8b9eb24f412dffc855c7bd854a8ecb3dff5@rizon.nodejumper.io:26656,92cd9bce4ec61a604ed7f7939105f483b68ea048@194.163.138.18:26656,b0e0bc65b4e9536eb56be4234e13f89c948a7c00@95.216.53.29:26656,f4147035f0cf892e942fe90a30bfc11a5b79bbea@168.119.240.200:47656,c55405a7a3ec1c6e1a893da7b3b482c5a74510df@173.249.44.140:26656,f9709f8c600c0e4a8a94cdb459e5ffe6d995a846@135.181.141.47:27656,14d18fef71c49a2950fba8ee25dc7702e1617010@65.108.108.42:26656,d208e886b3cee6474e48e134b3b4eca021d0e3c7@135.181.92.35:26656,2c0c07397755173d0aac6b927b981c9c12cafee4@135.181.96.158:26656,87b7bf580d08a069215f1dfc673a04b8bea3c437@65.21.94.220:16656,bfb77f894a951b467ffe1b1582a80054365b74ed@3.34.202.70:26656,b31ca1aa2cdf94b67630db1aaaf9a0ae64e44de7@65.108.130.189:26546,7437502688c0a088ecbfea277208f4eea1559e19@3.19.28.197:26656,36bc15ab8c1da911c14600d744b55e4af1ad8465@65.21.192.108:3090,35894de11b6d9ad326e1b7e82597dc89fafb3a5c@141.95.104.132:26656,0c6945b9dabd07f876f6ceb49888f22d34836d63@65.108.71.166:40656,6215c75f7353cccf226c7f7dbf3e173e4ecf7493@65.108.229.163:26656,150ab30efa957f4d78870cfb71c35fdd39cadd92@173.208.208.218:26656,83132dbfc5d2ffefb281434bd4aaef2f22439c5f@167.86.81.247:26656,4c3c0db4f660754132b543e1e4f20c648fd09525@136.49.122.190:26656,dc1f7fd216e1744caf4528386c5f6523e5ceeafa@65.108.98.228:26656,bb9c13f674f0f6579204c46be64802941ce26b40@65.108.11.161:36656,b3552be6ea1253410ebdc196ea2ad58422a3d319@31.7.207.16:40056,eca9124949dbb5155662eed91279760398fee00a@157.230.19.169:26656,ae1476777536e2be26507c4fbcf86b67540adb64@3.38.110.37:26656,71723b7f68af6570faf3b4745c8ce7432fd71c6e@142.132.152.187:11256,6ed9f2bf392b8c5549038007b80b12c500b105bd@51.195.234.88:2818,83c9cdc2db2b4eff4acc9cd7d664ad5ae6191080@3.38.142.63:26656,d89d96fd7aa1d5a184e2133d23dccb77f4a2e7a2@152.32.133.115:26676,4cfa1b8dfaf769285e5b8ee50b1b6565ba377901@13.125.254.28:26656,8cf465f058a9149f93f17fbb2b78a970e28214c6@65.108.140.2:26656,e08af7a66001edcc333d9b81398c257a9848d9a0@3.37.198.87:26656,ba73927a073d15fe0cbb80d7ff8660f72e73a492@3.38.109.37:26656,5852e539885c739cb36358e153c4025eb64fb01b@178.154.215.8:26656,426ea40a314df2879b6df1353e602137bcd69db7@65.21.88.252:28656,1a4ae2676b35c8f69e05dc5a89481540ca1470b7@195.201.106.36:26456,a753c4006779083aa8e9f6151bcf905351d85482@167.235.74.214:26656,443cc554b0d66f56082a490888ba81d972d47795@46.226.128.217:27656,6892d93153d9b13547982c6dc8d741084b599cea@3.210.29.92:26656,46a93373cf2720b077bd5b54e618d2db2685db94@144.126.136.37:26656,7fbf2f466df738891239b3c6a63898158ff1ef53@78.46.109.249:26656,e9a67088e89350acad50d89a521413605e942674@95.217.224.124:26666,8bb5c158e167305517733efda61082761ef9f555@135.181.57.209:24456,4b0a5f620970d6483c891d0883556155475d2fd9@138.201.30.152:26756,3dc3dc5954fb91bd790e8f307bd2a15adb2cc4db@217.79.252.58:26656,93c18ea1e84a3cacff6a861b350bf471b0638886@49.12.132.166:22256,3a99abdb437cf94ad755d96dcff328f32e68d282@65.21.136.58:56656,2801f595ccfb3cbed6d0fac55b2ec592c07b25d1@173.249.29.13:26656,2f1d8dd43d4ddb2b13b09511a0c18c90bfea534b@135.181.221.21:26656,81e55181d61a8d302b2f2f0a7720868fcbf7aa3f@52.78.54.7:26656,738e43d0adcf83060b298c45dcfaa5dd80014387@83.149.119.145:26656,e5aa800dbe1c1aa79caac701c69a7bdec5658262@8.219.137.18:26656,19f512844ed468330e868a5f436b188b5d0e62fa@5.161.54.226:26651,23dceeadd80dc99a59643bc7483d320c75f8b616@185.163.64.143:26656,3bd2cd72b827eba23009ae789d54aaf928c1f40d@3.37.16.163:26656,f30943bbc28bd3b94222b9749aa4b940f846b298@65.108.202.213:26656,8abf316257a264dc8744dee6be4981cfbbcaf4e4@3.36.42.3:26656,69d9dac0e7fe1e130e15016dd16be4b6f94f74c2@95.216.242.158:26766,60fb4dc175cbfd494f5e22c30ef1ae828ba26bfd@75.119.153.230:26656,5ae9b4d09f24b958720db03884f3687735bb6c60@15.164.219.113:26656,9b51b91fda0d1785fa03b4af1c3e36e53710850e@54.71.212.53:26656,6f50c0ac8df2586d3e8874f0043044b162378b40@52.78.116.1:26656,886a265607bd403a9646b5a3630f781d2d002478@159.69.171.132:26656,0c71edcd2f445679a9133bd8f59f65aa8d64852c@94.250.203.6:26676,8773ea296b9a68365af6a1e036ea6afdbeacf995@144.76.63.67:26109,e6433b244e34eb908d78c7a8cc864886c8e3c222@89.147.108.106:26656,d3ca96e62a701823195f345242b05816cfaeef9b@50.18.50.12:26656,3a78a701c045b40f2ba32fafd08fd6bd61e98c50@185.190.142.249:26656,af160a23a4070c3c2a1442a1fcb8937deb3c823f@18.216.106.203:26656,08d403fdb5c1862452fc4ddbf361007e1f95ad37@159.69.159.234:26656,4fae0139f072873e52b383a6e40534b2430eb1ae@31.7.196.9:26656,21c4d3d02746eaadf06afe9378f0dcf4f7660dfb@176.191.97.120:26656,0085e1f076717a5a1ce689ad99e6f5a22848659c@138.201.128.228:26666,805aba78d4f722695a1c77286c54d832aa37f314@65.108.106.172:36656,83212d4a5f0f2c0e072bab9e080ef0b892bec874@51.79.250.218:26656,618e9cdbb2ed7b07180984ee6adfe640676bbe70@65.108.99.254:26656,4c624c28b4144ed680d36f5e600f8a0c20d00d2f@135.181.142.206:26656,98365caa7d02d2ca11e58dc74522d2aa601eaf28@162.55.98.45:26444,58051c2fa300fbf816cc8e14a9f6c2d0013e09c8@13.36.72.182:28756,ca34f3c6c97f62f3cbbb7d00afc064c251d2b2af@95.179.208.31:26656,12a96839dfa68697c5c9b9728a3556af68cbf88a@15.164.212.255:26656,5c9692e80f475dd3eec1dd1a146b64db7b34445c@35.84.105.189:26656,8e99fc05313df741a0a7d86244bf4ab26a6202f4@15.164.219.196:26656,31456276cb6b87237ea3160ae9e7bf6c7c0ca5af@65.108.138.66:46656,0b33857e2156d74e355180dc4db6fb2fd80638b0@146.19.24.184:26656,bbcc62acee2672bf0ff947bd9057015c7c1787a3@31.7.207.245:28856,d7485b803d9750e11d20c0246ec6df7328b7e552@65.21.201.244:26866,40ebbce225506dec222854c6025593ce14c11eae@13.125.242.1:26656,358bf6432cc57a8fcd7ccf165dab7fae8e15e0b1@118.70.186.130:30656,e96a7a56c2f05b52270854246761f6cc6c434287@13.209.212.78:26656,50435cc01c9629b31013bcee4bb7f2276bb6513c@3.115.224.9:26656,8a9940980c6cf5c8b9c7639286202d8cab615d61@54.177.81.121:26656,8c6926383d9891d5e66c425762527fe0bfc0db08@13.124.216.13:26656,77fc91330de5a3f91c0a75d1b4ffdfa41be49111@15.165.237.238:26656,8e5079b569265990ce7ab1ea4688ee9372c264f9@135.181.219.115:26656,2089139e906255cb477869363a6106f99b9c41ed@65.21.200.6:28656,f28efad2d76857c2606f00bee014346481734ea0@116.203.29.116:46668,7575e45f006f558f15d2dcd26b68de88d580dced@144.2.71.66:46668"
	sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.rizon/config/config.toml
    seeds=""
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.rizon/config/config.toml
    sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.rizon/config/config.toml
    sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.rizon/config/config.toml

### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.rizon/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.rizon/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.rizon/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.rizon/config/app.toml

### Indexer (optional)

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.rizon/config/config.toml

## State Sync
```python
SNAP_RPC="https://rizon.nodejumper.io:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="0d51e8b9eb24f412dffc855c7bd854a8ecb3dff5@rizon.nodejumper.io:26656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.rizon/config/config.toml

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.rizon/config/config.toml
rizond tendermint unsafe-reset-all --home /root/.rizon
sudo systemctl restart rizond && journalctl -u rizond -f -o cat
```
# Create a service file

	sudo tee /etc/systemd/system/rizond.service > /dev/null << EOF
	[Unit]
	Description=Rizon Node
	After=network-online.target
	[Service]
	User=$USER
	ExecStart=$(which rizond) start
	Restart=on-failure
	RestartSec=10
	LimitNOFILE=10000
	[Install]
	WantedBy=multi-user.target
	EOF

## Start
```bash
sudo systemctl daemon-reload &&
sudo systemctl enable rizond &&
sudo systemctl restart rizond && sudo journalctl -u rizond -f -o cat
```
## Create validator


    rizond tx staking create-validator \
    --amount 1000000uatolo \
    --from <walletName> \
    --commission-max-change-rate "0.1" \
    --commission-max-rate "0.2" \
    --commission-rate "0.05" \
    --min-self-delegation "1" \
    --details="" \
    --identity="" \
    --pubkey  $(rizond tendermint show-validator) \
    --moniker <moniker> \
    --fees 2000uatolo \
    --chain-id titan-1 -y


## Delete node
    sudo systemctl stop rizond && \
    sudo systemctl disable rizond && \
    rm /etc/systemd/system/rizond.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .rizon && \
    rm -rf rizon && \
    rm -rf $(which rizond)

