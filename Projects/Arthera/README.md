# Arthera Testnet guide


![Arther](https://github.com/obajay/nodes-Guides/assets/44331529/7dc86ed0-6e4d-42c2-93c5-61f6552ec8c2)

[WebSite](https://www.arthera.net/)\
[GitHub](https://github.com/artheranet) \
[DOCS](https://docs.arthera.net/validators/devnet/)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 500GB    |


### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## Install Docker
```python
. <(wget -qO- https://raw.githubusercontent.com/SecorD0/utils/main/installers/docker.sh)
```
## Pull the latest Arthera node image
```python
docker pull arthera/arthera-node:1.2.0-devnet.rc2
```
## Create a folder to hold your Arthera node data
```python
mkdir $HOME/arthera
```
## Create a wallet for your validator
```python
docker run -it -v $HOME/arthera:/data arthera/arthera-node:1.2.0-devnet.rc2 account new
```
`Enter a password for the new account`
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/4267626d-99ae-4c57-823d-6109bdab0c08)
```python
1. Copy the generated secret key file from $HOME/arthera/keystore/UTC-**** to your workstation. You will need it later on.
2. Remember the password
```
## Create the validator identity (STEP 6
```python
docker run -it -v $HOME/arthera:/data arthera/arthera-node:1.2.0-devnet.rc2  validator new
```
`Enter a password for the new account`
![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/544c1264-9de1-49e0-82dd-ac6d27b6413e)


```python
1. Copy the validator's Public key displayed above
2. Remember the password
```
## Register your validator
This step requires that you submit the validator's wallet address created in step `Create a wallet for your validator` and the Public Key created in `Create the validator identity` to Arthera, so we can delegate the required coins to your validator (the minimum stake of 100,000 AA).


## View your validator in the Arthera Wallet  (STEP 8)
While you wait for us to delegate the required coins to your validator, you can register your account it in the [Arthera wallet](https://wallet-test.arthera.net/).

- Configure MetaMask to connect to the Arthera Devnet (see [Network details](https://docs.arthera.net/validators/devnet/#network-details))
- Import you wallet private key created in step `Create a wallet for your validator` into Metamask
- Go to [https://wallet-dev.arthera.net](https://wallet-dev.arthera.net/)
- Login with Metamask and select the imported account
- Under the Account menu you should see your new validator (after we delegate the required coins to it)
### The next steps require that your validator is shown in the Arthera Wallet. If you don't see it, wait for us to delegate the required coins to it and try again.

## Create a password file
Run the command below and replace YOUR_PASSWORD with the password entered in step `Create the validator identity`
```python
echo "YOUR_PASSWORD" > $HOME/arthera/password
```
## Generate your node key

- Run the command below to generate your unique node key:
```python
docker run -it -v $HOME/arthera:/data arthera/arthera-node:1.2.0-devnet.rc2 p2p genkey /data/node.key
```
- Check to see if the key was generated successfully by running:
```python
docker run -it -v $HOME/arthera:/data arthera/arthera-node:1.2.0-devnet.rc2 p2p enodeurl /data/node.key
```
- You should see an output similar to the one below:
```python
enode://f2927b8b1bc1b05997acf60f713cda3c776300cdac52b80d8dbfb2b434ef0e7283f8d3ca891d832bab3927b03c52593e575d48d9916804af5bf66a75b5b1288a@127.0.0.1:6534
```
- The output above is your node's enode URL. You will need it later on for identifying your node in the Metrics dashboard.

## Download the latest Devnet genesis file
- Run the command below to download the `devnet3.g` genesis file:
```python
curl -o $HOME/arthera/genesis.g https://s3.eu-central-1.amazonaws.com/release.arthera.net/devnet.g
```
## Test your node
- Replace the following variables and run the command below to start your validator:

`YOUR_PUBLIC_IP` with your actual Public IP address.
`ID_FROM_STEP_8` with the Validator ID obtained from step 8. View your validator in the Arthera Wallet
`VALIDATOR_PUBLIC_KEY_FROM_STEP_6` with the Public key of your validator from step 6. Create the validator identity
```python
docker pull arthera/arthera-node:1.2.0-devnet.rc2
docker run --name arthera-validator -p 6535:6060 -v $HOME/arthera:/data arthera/arthera-node:1.2.0-devnet.rc2 \
  --nodekey /data/node.key \
  --devnet \
  --nat extip:YOUR_PUBLIC_IP \
  --validator.id ID_FROM_STEP_8 \
  --validator.pubkey "VALIDATOR_PUBLIC_KEY_FROM_STEP_6"
```

- Check the output to have the following info:

![изображение](https://github.com/obajay/nodes-Guides/assets/44331529/28a6c67b-0764-4e68-937b-041f819407c2)


- If you get messages like `New event` and `New block` it means your node is syncing and working properly.

- You can now stop the node with Ctrl+C, clean the output folder with rm -rf $HOME/arthera/arthera-node $HOME/arthera/chaindata and remove the docker container with docker container rm arthera-validator and move to the next step.

## Run your node
- Replace the following variables and run the command below to start your validator:

`YOUR_PUBLIC_IP` with your actual Public IP address.
`ID_FROM_STEP_8` with the Validator ID obtained from step 8. View your validator in the Arthera Wallet
`VALIDATOR_PUBLIC_KEY_FROM_STEP_6` with the Public key of your validator from step 6. Create the validator identity
```python
docker container rm arthera-validator # remove any existing containers just to be sure
docker pull arthera/arthera-node:1.2.0-devnet.rc2
docker run --name arthera-validator -d --restart unless-stopped -p 6535:6060 -v $HOME/arthera:/data arthera/arthera-node:1.2.0-devnet.rc2 \
  --nodekey /data/node.key \
  --devnet \
  --verbosity 5 \
  --nat extip:YOUR_PUBLIC_IP \
  --validator.id ID_FROM_STEP_8 \
  --validator.pubkey "VALIDATOR_PUBLIC_KEY_FROM_STEP_6"
```

- Check the node was started with `docker ps`:
```python
CONTAINER ID   IMAGE                                   COMMAND                  CREATED         STATUS        PORTS                                                                                                              NAMES
d6ac0f5fdf14   arthera/arthera-node:1.2.0-devnet.rc2   "arthera-node --data…"   2 seconds ago   Up 1 second   0.0.0.0:18545-18546->18545-18546/tcp, :::18545-18546->18545-18546/tcp, 0.0.0.0:6535->6060/tcp, :::6535->6060/tcp   heuristic_northcutt
```

# Troubleshooting
- Node is not syncing
`The most common problems are firewalls and incorrectly provided external IP address.`

- Cleanup node data and restart validator
Stop the doocker container with `docker container stop arthera-validator`
Remove the database with `rm -rf $HOME/arthera/arthera-node $HOME/arthera/chaindata`
Start the docker container with `docker container start arthera-validator`

- Database is corrupted
  
See [Cleanup node data and restart validator](https://docs.arthera.net/validators/devnet/#2-cleanup-node-data-and-restart-validator)
