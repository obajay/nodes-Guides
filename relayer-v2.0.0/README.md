# Set up a v2.0.0
(This guide was taken from here https://github.com/zukaman/relayer-v2.0.0 )
<br>

In current example we will learn how to set up IBC relayer between two cosmos chains

## Update system
```
     sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
     sudo apt install wget git make htop unzip -y
```
## Install GO 1.18.3 | ONE COMMAND
```
     cd $HOME && \
     ver="1.18.3" && \
     wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
     sudo rm -rf /usr/local/go && \
     sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
     rm "go$ver.linux-amd64.tar.gz" && \
     echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
     source $HOME/.bash_profile && \
     go version
```
## Make relayer home dir
```
     cd $HOME
     mkdir -p $HOME/.relayer/config
```

## Download go-v2 relayer
```
     git clone https://github.com/cosmos/relayer.git
     cd relayer && git checkout v2.0.0
     make install
 
```
## Make you own config for Relayer v2
```
     cd $HOME
     mkdir -p $HOME/.relayer/config
     
     MEMO=Your memo 
     
     KEY1=Name of your key
     KEY2=Name of your key
     
     IP1=IP your GAIA node
     PORT1=Port RPC your GAIA node

     IP2=IP you STRIDE node
     PORT2=RPC port of your STRIDE node
```

# NEXT - COPY ALL ONE COMAND TO TERMINAL
```

sudo tee $HOME/.relayer/config/config.yaml > /dev/null <<EOF
global:
    api-listen-addr: :5183
    timeout: 10s
    memo: $MEMO
    light-cache-size: 20
chains:
    GAIA:
        type: cosmos
        value:
            key: $KEY1
            chain-id: GAIA
            rpc-addr: http://$IP1:$PORT1
            account-prefix: cosmos
            keyring-backend: test
            gas-adjustment: 1.3
            gas-prices: 0.01uatom
            debug: false
            timeout: 10s
            output-format: json
            sign-mode: sync
            strategy:
            type: native
            version: ics20-1
            order: UNORDERED
    stride:
        type: cosmos
        value:
            key: $KEY2
            chain-id: STRIDE-TESTNET-2
            rpc-addr: http://$IP2:$PORT2
            account-prefix: stride
            keyring-backend: test
            gas-adjustment: 1.3
            gas-prices: 0.01ustrd
            debug: false
            timeout: 20s
            output-format: json
            sign-mode: sync
            strategy:
            type: native
            version: ics20-1
            order: UNORDERED
paths:
    task:
        src:
            chain-id: STRIDE-TESTNET-2
            client-id: 07-tendermint-0
            connection-id: connection-0
            port-id: icacontroller-GAIA.DELEGATION,transfer,icacontriiler-GAIA.FEE,icacontroller-GAIA.WITHDRAWAL,icacontroller-GAIA.REDEMPTION
  
        dst:
            chain-id: GAIA
            client-id: 07-tendermint-0
            connection-id: connection-0
            port-id: icahost,icacontroller-GAIA.DELEGATION,transfer,icacontriiler-GAIA.FEE,icacontroller-GAIA.WITHDRAWAL,icacontroller-GAIA.REDEMPTION
        src-channel-filter:
            rule: allowlist
            channel-list:
                - channel-0 
                - channel-1 
                - channel-2 
                - channel-3 
                - channel-4 
        dst-channel-filter:
            rule: allowlist
            channel-list:
                - channel-0 
                - channel-1 
                - channel-2 
                - channel-3 
                - channel-4
EOF
```

## Import  keys for the relayer to use when signing and relaying transactions
   ```
     rly keys restore stride $KEY2 "mnemonic words here"
     rly keys restore GAIA $KEY1 "mnemonic words here"
   ```
## Create go-v2 relayer service file
 (copy and paste into the terminal with one command)
```
sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
[Unit]
Description=Relayer_v2
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which rly) start task --log-format logfmt --processor events
Restart=on-failure
RestartSec=10
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF
```

## Start service
```
     sudo systemctl daemon-reload
     sudo systemctl enable rlyd
     sudo systemctl restart rlyd && journalctl -u rlyd -f
```
