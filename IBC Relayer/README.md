# IBC Relayer Hermes
## For example Stride & Gaia
![inter blockchain communication](https://user-images.githubusercontent.com/44331529/182154661-6e71ef48-eeda-45e5-9b55-17a4a544260f.png)
![chain a chain b](https://user-images.githubusercontent.com/44331529/182154667-5a57b686-ae64-400f-af31-cb1c648d5939.jpg)
=
## about [Hermes](https://hermes.informal.systems/index.html)

## Preparing
The Relayer can be installed on the same server along with the necessary nodes, or on a separate server if an open RPC nodes is known. \
The guide will describe the option when both Hermes and both nodes are installed on the same server.
+ Install both nodes. In this case, we are interested in the node - `Stride (STRIDE-TESTNET-2)` and  `Cosmos (GAIA)`
+ Nodes must be in sync with the network
+ Setting the minimum commission for gas (minimum-gas-prices = "0uatom" / minimum-gas-prices = "0ustrd")  (app.toml)
+ Pre-open the RPC for each of the nodes (config.toml line 91 0.0.0.0:26657)
+ Indexing should be set to position --> indexer = "kv"  (config.toml)
+ Check if relaying is supported
```console
strided q ibc-transfer params
```
`receive_enabled: true` \
`send_enabled: true`
```console
gaiad q ibc-transfer params
```
`receive_enabled: true` \
`send_enabled: true`

## Create .hermes folder
```console
cd $HOME
mkdir -p $HOME/.hermes/bin
```
## Install dependencies
```console
sudo apt update && sudo apt upgrade -y
sudo apt install unzip -y
```
## Install software
```console
wget "https://github.com/informalsystems/ibc-rs/releases/download/v0.15.0/hermes-v0.15.0-x86_64-unknown-linux-gnu.tar.gz"
tar -C $HOME/.hermes/bin/ -vxzf hermes-v0.15.0-x86_64-unknown-linux-gnu.tar.gz
rm hermes-v0.15.0-x86_64-unknown-linux-gnu.tar.gz

echo "export PATH=$PATH:$HOME/.hermes/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Installing configurations
### STRIDE CHAIN
```console
MEMO_PREFIX="ENTER_YOUR_RELAYER_NAME"
STRIDE_RPC="127.0.0.1"
STRIDE_RPC_PORT="21017"
STRIDE_GRPC_PORT="9986"
STRIDE_CHAIN_ID="STRIDE-TESTNET-2"
STRIDE_ACC_PREFIX="stride"
STRIDE_DENOM="ustrd"
STRIDE_WALLET="ENTER_YOUR_WALLET_NAME"
```
### GAIA CHAIN
```console
GAIA_RPC="127.0.0.1"
GAIA_RPC_PORT="23657"
GAIA_GRPC_PORT="23090"
GAIA_CHAIN_ID="GAIA"
GAIA_ACC_PREFIX="cosmos"
GAIA_DENOM="uatom"
GAIA_WALLET="ENTER_YOUR_WALLET_NAME"
```
### Save variables
```console
echo "
export STRIDE_CHAIN_ID=${STRIDE_CHAIN_ID}
export STRIDE_DENOM=${STRIDE_DENOM}
export STRIDE_WALLET=${STRIDE_WALLET}
export GAIA_CHAIN_ID=${GAIA_CHAIN_ID}
export GAIA_DENOM=${GAIA_DENOM}
export GAIA_WALLET=${GAIA_WALLET}
export MEMO_PREFIX=${MEMO_PREFIX}
" >> $HOME/.bash_profile

source $HOME/.bash_profile
```
## Create config  (Copy everything and paste)  
```console
echo "[global]
log_level = 'info'
[mode]
[mode.clients]
enabled = true
refresh = true
misbehaviour = true
[mode.connections]
enabled = false
[mode.channels]
enabled = false
[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = true
[rest]
enabled = true
host = '0.0.0.0'
port = 3000
[telemetry]
enabled = true
host = '0.0.0.0'
port = 3001
[[chains]]
id = '$STRIDE_CHAIN_ID'
rpc_addr = 'http://$STRIDE_RPC:$STRIDE_RPC_PORT'
grpc_addr = 'http://$STRIDE_RPC:$STRIDE_GRPC_PORT'
websocket_addr = 'ws://$STRIDE_RPC:$STRIDE_RPC_PORT/websocket'
rpc_timeout = '10s'
account_prefix = '$STRIDE_ACC_PREFIX'
key_name = '$STRIDE_WALLET'
store_prefix = 'ibc'
max_tx_size = 100000
max_gas = 20000000
gas_price = { price = 0.001, denom = '$STRIDE_DENOM' }
gas_adjustment = 0.1
max_msg_num = 15
clock_drift = '5s'
trusting_period = '8hours'
memo_prefix='$MEMO_PREFIX'
trust_threshold = { numerator = '1', denominator = '3' }
[[chains]]
id = '$GAIA_CHAIN_ID'
rpc_addr = 'http://$GAIA_RPC:$GAIA_RPC_PORT'
grpc_addr = 'http://$GAIA_RPC:$GAIA_GRPC_PORT'
websocket_addr = 'ws://$GAIA_RPC:$GAIA_RPC_PORT/websocket'
rpc_timeout = '10s'
account_prefix = '$GAIA_ACC_PREFIX'
key_name = '$GAIA_WALLET'
store_prefix = 'ibc'
max_tx_size = 100000
max_gas = 30000000
gas_price = { price = 0.001, denom = '$GAIA_DENOM' }
gas_adjustment = 0.1
max_msg_num = 15
clock_drift = '5s'
trusting_period = '8hours'
memo_prefix= '$MEMO_PREFIX'
trust_threshold = { numerator = '1', denominator = '3' }" > $HOME/.hermes/config.toml

```

## Output result
```console
hermes config validate
```
![1dsd](https://user-images.githubusercontent.com/44331529/182159481-28d8a5ce-f320-4600-9891-19b830e7e94e.png)
```console
hermes health-check
```
![swe](https://user-images.githubusercontent.com/44331529/182160098-1cdfd508-d3f0-4dca-8402-3d2d285b4a0d.png)


## Create wallets or restore your wallet with mnemonic
### Stride Wallet
```
strided keys add $STRIDE_WALLET
```
(please save mnemonic and wallet address)
### Gaia wallet
```
gaiad keys add $GAIA_WALLET
```
(please save mnemonic and wallet address)
## Add wallet to Hermes
### Add wallet Stride
```console
hermes keys restore $STRIDE_CHAIN_ID -n $STRIDE_WALLET -m "your mnemonic phrase"
```
### Add wallet Gaia
```console
hermes keys restore $GAIA_CHAIN_ID -n $GAIA_WALLET -m "your mnemonic phrase"
```
## Connecting to the channel
### Enter vars channel from output
```console
HERMES_STRIDE_GAIA_CHANNEL_ID="channel-0"
HERMES_GAIA_STRIDE_CHANNEL_ID="channel-0"

echo "
export HERMES_STRIDE_GAIA_CHANNEL_ID=${HERMES_STRIDE_GAIA_CHANNEL_ID}
export HERMES_GAIA_STRIDE_CHANNEL_ID=${HERMES_GAIA_STRIDE_CHANNEL_ID}
" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Run the service file
```console
sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
[Unit]
Description=HERMES
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which hermes) start
Restart=on-failure
RestartSec=10
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF
```
### Restart Hermes for start and run logs
```console
sudo systemctl daemon-reload
sudo systemctl enable hermesd
sudo systemctl restart hermesd && journalctl -u hermesd -f
```
![ssd2 (1)](https://user-images.githubusercontent.com/44331529/182162985-5a35b3a3-1be6-4113-8e92-4dc9cdaab346.png)

![ssd2 (2)](https://user-images.githubusercontent.com/44331529/182163008-95100b00-ee30-4e20-9d9a-3dae1f8841d1.png)

### You can send tokens if you want

    strided tx gaiad  tx ibc-transfer transfer transfer channel-0 YOUR_WALLET_ADDRESS_GAIA 500uatom --from=GAIA_WALLET --fees 500uatom channel-0  YOUR_WALLET_ADDRESS_STRIDE 500ustrd --from=STRIDE_WALLET --fees 500ustrd
    gaiad  tx gaiad  tx ibc-transfer transfer transfer channel-0 YOUR_WALLET_ADDRESS_GAIA 500uatom --from=GAIA_WALLET --fees 500uatom channel-0 YOUR_WALLET_ADDRESS_GAIA 500uatom --from=GAIA_WALLET --fees 500uatom

