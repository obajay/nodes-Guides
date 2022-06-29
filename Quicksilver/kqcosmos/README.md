    sudo apt update && sudo apt upgrade -y
    sudo apt install curl build-essential git wget jq make gcc tmux nvme-cli -y

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

    git clone https://github.com/ingenuity-build/interchain-accounts --branch main
    cd interchain-accounts
    make install

    icad init <name_node> --chain-id kqcosmos-1
    icad keys add <name_wallet> --recover

    wget -O $HOME/.ica/config/genesis.json "https://raw.githubusercontent.com/ingenuity-build/testnets/main/killerqueen/kqcosmos-1/genesis.json"
    sed -i.bak -e "s/^minimum-gas-prices =./minimum-gas-prices = "0.00uqck"/;" ~/.ica/config/app.toml

    seeds="66b0c16486bcc7591f2c3f0e5164d376d06ee0d0@65.108.203.151:26656"
    sed -i.bak -e "s/^seeds =.*/seeds = "$seeds"/" $HOME/.ica/config/config.toml

    sudo tee /etc/systemd/system/icad.service > /dev/null <<EOF
    [Unit]
    Description=ica
    After=network-online.target
    
    [Service]
    User=$USER
    ExecStart=$(which icad) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF 

    sudo systemctl daemon-reload && \
    sudo systemctl enable icad && \
    sudo systemctl restart icad && sudo journalctl -u icad -f -o cat

    curl localhost:26657/status

    icad tx staking create-validator \
      --amount 10000000uatom \
      --from wallt name \
      --commission-max-change-rate "0.01" \
      --commission-max-rate "0.2" \
      --commission-rate "0.07" \
      --min-self-delegation "1" \
      --pubkey  $(icad tendermint show-validator) \
      --moniker Moniker \
      --chain-id kqcosmos-1 \
      --node grep -oPm1 "(?<=^laddr = \")([^%]+)(?=\")" $HOME/.ica/config/config.toml -y
