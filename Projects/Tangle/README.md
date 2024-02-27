# Tangle Testnet guide

![tangl](https://github.com/obajay/nodes-Guides/assets/44331529/78b39cb8-33de-4b55-ba66-852527e5a29d)


[WebSite](https://www.tangle.tools/)\
[GitHub](https://github.com/webb-tools/tangle/blob/d12e316d7c7f64fdcc76842e03944a43221285d8/chainspecs/testnet/tangle-standalone.json)
=
[EXPLORER](https://explorer.tangle.tools/?ref=blog.webb.tools) \
[TELEMETRY](https://telemetry.polkadot.io/#list/0x3d22af97d919611e03bbcbda96f65988758865423e89b2d99547a6bb61452db3) \
[CHAIN](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Ftestnet-rpc.tangle.tools#/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

# Build 10.01.24
```python
cd $HOME
mkdir -p $HOME/.tangle && cd $HOME/.tangle
wget -O tangle https://github.com/webb-tools/tangle/releases/download/v0.6.1/tangle-testnet-linux-amd64 && chmod +x tangle
mv tangle /usr/bin/
```

`tangle --version`
- tangle 0.6.1-721ffa6-x86_64-linux-gnu


# Create a service file
```python
yourname=<name>
```
```python
sudo tee /etc/systemd/system/tangle.service > /dev/null << EOF
[Unit]
Description=Tangle Validator Node
After=network-online.target
StartLimitIntervalSec=0
[Service]
User=$USER
Restart=always
RestartSec=3
ExecStart=/usr/bin/tangle \
  --base-path $HOME/.tangle/data/validator/$yourname \
  --name '$yourname' \
  --chain tangle-testnet \
  --auto-insert-keys \
  --node-key-file "/home/$yourname/node-key" \
  --port 30333 \
  --rpc-port 9933 \
  --prometheus-port 9515 \
  --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
  --validator \
  --pruning archive \
  --no-mdns
[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
systemctl daemon-reload
systemctl enable tangle
systemctl restart tangle && journalctl -u tangle -f -o cat
```

- After launch, we wait for our node to synchronize. You can track our condition using telemetry

[TELEMETRY](https://telemetry.polkadot.io/#list/0x3d22af97d919611e03bbcbda96f65988758865423e89b2d99547a6bb61452db3)
=

- After the node has synchronized, we pull out the key from our node by entering the command
```python
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```

## BACKUP
🟢 save the located keys in `$HOME/.tangle/node-key` and `$HOME/.tangle/data/validator/YOURNAME/chains/tangle-testnet/keystore/`

## Creating a validator
- Go to the [website](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Ftestnet-rpc.tangle.tools#/accounts) and first create a wallet
- We create a validator. To do this, select `Network - Staking - Accounts - Validator`

`logs`
```python
journalctl -u tangle -f -o cat
```
`restart`
```
systemctl restart tangle && journalctl -u tangle -f -o cat
```
`delete node`
```
systemctl stop tangle &&
systemctl disable tangle &&
rm /etc/systemd/system/tangle.service &&
systemctl daemon-reload &&
cd
rm -r .tangle
```
