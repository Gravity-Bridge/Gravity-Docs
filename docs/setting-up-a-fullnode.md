# How to run a Gravity testnet full node

A Gravity chain full node is just like any other Cosmos chain.
Unlike the validator flow no external software is required.

## What do I need

A Linux server with any modern Linux distribution, 2gb of ram and at least 20gb storage.q

### Download Gravity chain software

```bash
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.0/gravity-v0.1.5-linux-amd64
mv gravity-v0.1.5-linux-amd64 gravity

chmod +x gravity
sudo mv gravity /usr/bin/
```

### Init the config files

```bash
cd $HOME
gravity init mymoniker --chain-id gravity-bridge-test1
```

### Copy the genesis file

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/gravity-docs/main/genesis.json
cp genesis.json $HOME/.gravity/config/genesis.json
```

### Add seed node

Change the seed field in ~/.gravity/config/config.toml to contain the following:

```text

seeds = "6a9cd8d87ab9e49d7af91e09026cb3f40dec2f85@gravity-chain.althea.net:26656"

```

### Start your full node and wait for it to sync

```bash
gravity start
```

# Firewall Settings

This is optional but greatly helps with the health of the network. 

## Open the inbound P2P port
```
sudo ufw allow from any to any port 26656 proto tcp
```
