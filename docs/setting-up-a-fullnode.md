# How to run a Althea testnet full node

A Althea chain full node is just like any other Cosmos chain.
Unlike the validator flow no external software is required.

## What do I need

A Linux server with any modern Linux distribution, 2gb of ram and at least 20gb storage.q

### Download Althea chain software

```bash
# the althea chain binary itself
wget https://github.com/althea-net/althea-chain/releases/download/v0.2.3/althea-0.2.2-18-g73447b6-linux-amd64
mv althea-0.2.2-18-g73447b6-linux-amd64 althea

chmod +x althea
sudo mv althea /usr/bin/
```

### Init the config files

```bash
cd $HOME
althea init mymoniker --chain-id althea-testnet2v3
```

### Copy the genesis file

```bash
wget https://github.com/althea-net/althea-chain/releases/download/v0.2.3/althea-testnet2v3-genesis.json
cp althea-testnet2v3-genesis.json $HOME/.althea/config/genesis.json
```

### Add seed node

Change the seed field in ~/.althea/config/config.toml to contain the following:

```text

seeds = "6a9cd8d87ab9e49d7af91e09026cb3f40dec2f85@testnet2.althea.net:26656"

```

### Start your full node and wait for it to sync

Ask what the current blockheight is in the chat

```bash
althea start
```
