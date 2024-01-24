# Osmosis Mainnet guide

![osmo](https://user-images.githubusercontent.com/44331529/195998983-0dc2ca0d-7192-471f-95fa-67ee8f467a54.png)

[WEBSITE](https://app.osmosis.zone/) \
[GitHub](https://github.com/osmosis-labs/osmosis)
=
[EXPLORER 1](https://www.mintscan.io/osmosis/validators) \
[EXPLORER 2](https://explorer.stavr.tech/Osmosis-Mainnet) \
[EXPLORER 3](https://ping.pub/osmosis/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  16| 32GB | 500GB    |


# 1) Auto_install script
```python
wget -O osmo https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Osmosis/osmo && chmod +x osmo && ./osmo
```

# 2) Manual installation

### Preparing the server

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.21.4
```python
ver="1.21.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Build 18.01.24
```python
cd $HOME
git clone https://github.com/osmosis-labs/osmosis && cd osmosis
git checkout v22.0.0
make install
```

*******🟢UPDATE🟢******* 18.01.24
```python
cd $HOME
wget -O osmosisd https://github.com/osmosis-labs/osmosis/releases/download/v22.0.0/osmosisd-22.0.0-linux-amd64
chmod +x osmosisd
mv osmosisd $(which osmosisd)
osmosisd version --long
# version: 22.0.0
# commit: 60ee947d50408a4273013422bce5acba19deb63f
systemctl restart osmosisd && journalctl -u osmosisd -f -o cat
```

`osmosisd version --long | head`
- version: 22.0.0
- commit: 60ee947d50408a4273013422bce5acba19deb63f

```python
osmosisd init STAVR_guide --chain-id osmosis-1
```    

## Create/recover wallet
```python
osmosisd keys add <walletname>
      OR
osmosisd keys add <walletname> --recover
```

## Download Genesis

```python
wget -O $HOME/.osmosisd/config/genesis.json "https://github.com/osmosis-labs/networks/raw/main/osmosis-1/genesis.json"
```
`sha256sum ~/.osmosisd/config/genesis.json`
+ 1cdb76087fabcca7709fc563b44b5de98aaf297eedc8805aa2884999e6bab06d

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uosmo\"/;" ~/.osmosisd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.osmosisd/config/config.toml
peers="83c06bc290b6dffe05aa9cec720bedfc118afcbc@osmosis.nodejumper.io:35656,f95d9634ad68b8f0ac80ce308adb71d8c119ada5@141.98.219.104:26656,01ce9f04c0293a3c4fb28006c526284eccfc59a7@144.76.117.155:26656,46f081e10f15eb5f3d1aa648b46598413133c506@54.255.207.239:26656,e3a17384a87cfca72b2cb5a0d642d1192fb3749e@65.108.110.206:26656,10d3bc0ddc69c0b2068f2785f7d0e135e2f99d54@142.132.194.239:26656,2736d870197d443e463b4ff4b7b52f1cec920030@45.63.39.14:26656,89db0da8469e35d62a719dbfe93a2400ce403557@198.244.176.186:56656,7270986f287d172fe000657ace73154859e95ca7@35.215.11.66:26656,f1ef1529431415d27945de3ac51f865c7d0e0d5d@65.21.230.104:26626,c879d94349a7be8168166250c084c68698a1d18d@45.32.181.106:26656,11de40c9740e652c7b1993a142a7dee62ffdccb5@104.156.252.98:26656,089b0de9671dc3cd00ded782693c03509b78b5d9@43.201.25.59:26656,3fd77551458331b433c23809270f555663c24bfe@65.109.28.58:26656,d1b50d00892c0f0d41241f377a49ae3bf731211c@18.191.220.170:26656,74e8ba742d8312c250f3237c8c8f3f951c01f9df@95.216.4.104:2003,7f246e899935b4079d053382ffe43b6d91554984@162.19.83.194:26646,980b15331dece2aa8020c1800b9c00ddb273c872@138.201.32.103:30656,56fa1755d27cb5d8b061d5beeb0a969054ce056e@43.153.101.118:26656,d2a4b47f8d2f4130943ef7a38c1c1c80ea4d06fd@199.247.29.239:26656,9755cab2585a2794453a5b396ef13b893393366f@65.108.212.224:46655,e613079d9b1c1c688963215a975cc9b29722f4fb@65.108.238.103:12556,fc590afe489a1b9ca8ff3f2fb396dbc20b1997a4@204.16.244.254:26656,e23e9a19bd699ab3536db0e3c174548288cf77fd@35.158.189.143:26656,626f1a86b94b3e48c08413a7af817d0f9e9985f2@142.132.213.115:26656,a0de522673ebc9545bc58bffac333d372ff81d2d@185.156.44.76:26656,01e80a29f702c431f1f5bbac8b4d3d2963d985b7@65.108.14.96:56656,3e2687691d1e7c82b5880b55d2ea13e5d517b9fe@3.139.178.87:26656,95f7624ab26311f8774eb2452577fed9ce4a1efa@5.9.87.218:26656,b8450ac06ab8ccac21b21bbbba8ea3751a479291@52.91.239.243:26656,244da65489afc840d85aeb7bc9c7c6e9ec094c52@3.35.205.240:26656,8e49e857d678cc2488a370474ffd9ea817982048@65.109.21.209:26656,7de231d5c75feb810a9196fa2a3e83e0576c88a9@212.95.53.152:26656,7c5459ea4bbc41aa4d86ffe8126f0651155227c8@85.195.102.127:26656,31e7a8b8cc97e85472c609f9d220fdd9536d4f4d@94.130.220.54:26656,3197daa0ee5245b17a546be032ff0f6814e1d1db@148.251.191.239:26656,21934d3bc6121c61ad56bf44fbe78dd144bde4d3@54.213.197.155:26656,82e224c9640048a6513c589e904c0d903bb99f32@74.118.140.23:26656,8e5051244ada217dc580feb774789ad7eb8b357c@34.135.79.149:26656,9c7174051aa1ba0c90340ee9c5433d2aae4c3d28@67.209.54.22:26656,2186d344ff775c8181bf31de600eed0c72b9fe9e@65.109.28.213:26656,407267ac44b20a0a4258d0bbca1c9f657bf88d08@74.118.143.19:26656,8cee4f3d7966b95b723a4a1c6e31e72e22cbe4a8@35.162.72.203:26656,42745690b41f6a7515c4a87d88efda2e82b55b76@78.46.94.183:26656,ff57203dd2ae45c0098257d1a1f2b313ce565b51@18.217.113.229:26656,aa88cb583b8d932cadfcfd40de6594a64200da93@167.235.135.248:26656,71f2451869d7363ce5d91366143de63069641303@65.108.71.166:33656,800c7d4af8c1d9beefaa37b3dfa94e9dd928a05e@141.98.218.27:26656,797094953d830f8727f3b5175f2b205df16d5867@45.77.212.231:26656,166c097feebc3c6b8c3ef2fd0ca168e9aaa505df@65.108.41.56:26656,abf9746aab3966d868cf74b0be687d641358afa0@3.37.44.181:26656,938d7d9d8f2a7ba1e2a48c8755ba1bf9d61ece51@52.66.169.123:26656,a6283307952423c1751431c220d11ed36b61ed84@143.110.237.113:26656,69e0d192ebbf4599bcb0679901c020cde07e7806@176.9.28.41:26656,5a037f6f26c730650e67ac655bf2c0031c188493@34.220.204.41:26656,0660d18b65340a55514f240dd517282ca286f169@176.9.28.62:26656,36e02a2eaa8e424df5ccf2d64f4f5bbb7e364842@51.89.192.131:26656,43785e5ffd8783393ea8094f77efcee5bdbcdce3@78.141.244.18:26656,2f5e471d41e03da1f0ae62507ec6280872e493fe@162.19.21.28:26656,b022cf434d838b0bc6135bfe45fb083c270233ce@65.21.204.171:36656,e891d42c31064fb7e0d99839536164473c4905c2@47.156.153.124:31656,9a67a02775215ad3901de09534db39ca476bc8ca@65.108.229.163:20656,038644cdab5548ab7c9e57784ce324181085d94c@23.88.67.24:10256,e81c3c20833cfb5d652a9c842c9f1c8b1835479d@108.61.190.21:26656,72cd15ffcfd844985ccd14789a163a986ef82471@52.48.78.18:26656,61a24557079f0b26e617f6d7df3ae17203b718b3@5.189.176.77:26656,ca0481d7013194692c586eb78081fa4f298c6ccf@3.98.124.32:26656,5372125cc22180420db51c79474702a47e7201f4@15.235.53.27:26656,a857364178b28eb5173c04e0de9a03f1c4b1d7a0@141.98.219.216:26656,20e1000e88125698264454a884812746c2eb4807@65.108.227.217:12556,b76068b52bffb03ea585938c747f65c27fd9714e@34.105.118.216:26656,c4c904c24818a0aa4f4216060d0b23f52dbe0c23@5.9.120.252:26656,4c4b7c32af3f6f723a1011e00281b321c5137aef@65.21.34.249:16190,f7c4d75fcca71038ffe2009c5eb50179f6a05071@164.92.78.10:26656,2e9f93da71e409dbb972610a9885e30c522c9d53@65.109.32.85:26656,15ba6c278c8692d08f2c64390db073021e7914bf@34.244.147.216:26656,689970f8ec726af4f78ae77dab14b94e95bc9fd5@209.34.206.36:26656,3d311ac374e6953e97ee07c74a76f399394c3025@173.215.85.171:20000,1876eb08c7e93c965a895177f82c8725f89c0f65@54.214.183.228:26656,1ca4c8d66a66580909f8ebe66a89be00d9a96cb8@136.244.29.116:27656,f14b97210391255b5b439ce70cac7ad6a4694cc0@3.35.197.22:26656,20913e92e8b9ea2d80ad34edd9b52e97886cf616@54.37.30.181:26656,63e4dd6530bf4dc4c2202be256b262a27d661106@157.90.5.101:26656,f4b811759e55f665180545ad5e1b42573f660861@135.181.181.251:26656,724cef11bbe866269b3d67f7dd5ea539cc4096bf@198.244.164.186:26656,9737b3116f87163124f8b5aeca1a6f8498f04f70@35.88.148.195:26656,4017c243549b8bb4ad2b4cfe5d685aea450dcbcd@209.34.206.35:26656,bfb67b2ae345955d6bc0991450120669c683386e@149.56.25.66:26656,efbe79e244693ae343dd8b6308bf4b708da82200@74.118.139.212:26656,79903bcd0776cb5b35511c9697c92b2b8e50262a@15.235.53.26:26656,fd0930fea06876e362e0a92046854ed651f27ac2@45.76.13.41:26656,ed122e68a8887fc44ce24b1da680c572bf37cc3a@216.128.183.5:26656,68a7fb268213f0b8bb498fcb97cb309a3e611da7@52.66.232.98:26656,807eda3abecff79df294d127cf58d6d5e07393ee@67.209.54.21:26656,5c3b41ae76748d36b738b8e63a72e0ef66b92b4f@167.99.255.25:26656,2904827f3ffa642bf7122d65cef27e1ab40a7346@35.74.104.174:16656,4a856d135a431039918b8a2febd4e415eb2b7cc3@43.153.96.33:26656,a5a4a5abe197153c9414e5fbf3a817305839aa84@35.79.41.69:26656,d49fe7ffb8d6ec35525279db8ede63017d5cdf5d@51.222.42.203:26656,6178f129efa76d235436e2156959d0acb4772c6a@65.108.128.168:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.osmosisd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.osmosisd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.osmosisd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.osmosisd/config/config.toml

```
### Pruning (optional)
```bash
pruning="custom" &&
pruning_keep_recent="100" &&
pruning_keep_every="0" &&
pruning_interval="10" &&
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.osmosisd/config/app.toml &&
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.osmosisd/config/app.toml &&
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.osmosisd/config/app.toml &&
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.osmosisd/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" &&
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.osmosisd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.osmosisd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Osmosis/addrbook.json"
```

[SNAPSHOT](https://polkachu.com/tendermint_snapshots/osmosis)
=

# Create a service file
```python
sudo tee /etc/systemd/system/osmosisd.service > /dev/null <<EOF
[Unit]
Description=osmosisd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which osmosisd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable osmosisd
sudo systemctl restart osmosisd && sudo journalctl -u osmosisd -f -o cat
```

### Create validator
```python
SOON
```

## Delete node
```python
systemctl stop osmosisd
systemctl disable osmosisd
rm /etc/systemd/system/osmosisd.service
systemctl daemon-reload
cd $HOME
rm -rf .osmosisd osmosis
rm -rf $(which osmosisd)
```
