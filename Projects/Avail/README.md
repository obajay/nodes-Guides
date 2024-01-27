# Avail Testnet guide


![avail](https://github.com/obajay/nodes-Guides/assets/44331529/d3f953c2-59c8-4e77-8204-ead225b79674)


[WebSite](https://www.availproject.org/#overview)\
[GitHub](https://github.com/availproject/avail)
=
[TELEMETRY](https://telemetry.avail.tools/#/0x6f09966420b2608d1947ccfb0f2a362450d1fc7fd902c29b67c906eaa965a7ae) \
[CHAIN](https://goldberg.avail.tools/#/staking) \
[LeaderBoard](https://leaderboard.availproject.org/)
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

# Build 16.01.24
```python
wget https://github.com/availproject/avail/releases/download/v1.9.0.0/x86_64-ubuntu-2204-data-avail.tar.gz
tar xvzf x86_64-ubuntu-2204-data-avail.tar.gz
rm -rf x86_64-ubuntu-2204-data-avail.tar.gz
mv data-avail /usr/bin/avail
```


# UPDATE 16.01.24
```python
cd $HOME
wget https://github.com/availproject/avail/releases/download/v1.9.0.0/x86_64-ubuntu-2204-data-avail.tar.gz
tar xvzf x86_64-ubuntu-2204-data-avail.tar.gz
rm -rf x86_64-ubuntu-2204-data-avail.tar.gz
mv data-avail $(which avail)
avail --version
#avail 1.9.0-unknown
systemctl restart avail && journalctl -u avail -f -o cat
```

`avail --version`
- avail 1.9.0-unknown

# Create a service file
```python
yourname=<name>
```
```python
tee /etc/systemd/system/avail.service > /dev/null << EOF
[Unit]
Description=Avail Validator Node
After=network-online.target
StartLimitIntervalSec=0
[Service]
User=$USER
Restart=always
RestartSec=3
LimitNOFILE=65535
ExecStart=/usr/bin/avail \
  --base-path $HOME/.avail/data/ \
  --chain goldberg \
  --port 30333 \
  --rpc-port 9933 \
  --prometheus-port 9615 \
  --validator \
  --name '$yourname'
[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
systemctl daemon-reload
systemctl enable avail
systemctl restart avail && journalctl -u avail -f -o cat
```

- After launch, we wait for our node to synchronize. You can track our condition using telemetry

[TELEMETRY](https://telemetry.avail.tools/#/0x6f09966420b2608d1947ccfb0f2a362450d1fc7fd902c29b67c906eaa965a7ae)
=

- After the node has synchronized, we pull out the key from our node by entering the command
```python
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```

## BACKUP
🟢 save the located keys in `$HOME/.tangle/node-key` and `$HOME/.tangle/data/chains/tangle-standalone-testnet/keystore/`

## Creating a validator
- Go to the [website](https://goldberg.avail.tools/#/explorer) and first create a wallet
- We create a validator. To do this, select `Network - Staking - Accounts - Validator`

`logs`
```python
journalctl -u avail -f -o cat
```
`restart`
```
systemctl restart avail && journalctl -u avail -f -o cat
```
`delete node`
```
systemctl stop avail
systemctl disable avail
rm /etc/systemd/system/avail.service
systemctl daemon-reload
cd
rm -r .avail
```
