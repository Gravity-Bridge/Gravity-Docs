# How to run a Gravity Bridge mainnet full node

A Gravity chain full node is just like any other Cosmos chain.
Unlike the validator flow no external software is required.

## What do I need

A Linux server with any modern Linux distribution, 2gb of ram and at least 20gb storage.

### Download Gravity chain software

Syncing starts with v1.4.0 and you should upgrade to v1.5.0,v1.5.2,v1.6.4, and v1.7.0 when prompted again

```bash
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.4.0/gravity-linux-amd64
mv gravity-linux-amd64 gravity

chmod +x gravity
sudo mv gravity /usr/bin/
```

### Init the config files

```bash
cd $HOME
gravity init mymoniker --chain-id gravity-bridge-3
```

### Copy the genesis file

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/gravity-docs/main/genesis.json
cp genesis.json $HOME/.gravity/config/genesis.json
```

### Add seed node and persistent peers

Change the `seeds` field in ~/.gravity/config/config.toml to contain the following:

```text
seeds = "6770e29a9224810bcde6655b742d52b8a49d51e8@65.19.136.133:26656,63e662f5e048d4902c7c7126291cf1fc17687e3c@95.211.103.175:26656,95f47663d4b49c9610325ecb4c9e1ffc511ccb26@gravity.mahdisworld.net:26656"

```

Change the `persistent_peers` field in ~/.gravity/config/config.toml to contain the following:
```text

persistent_peers = "73e27e9b376d2f58d80e29e8175542cb01c3024d@135.181.73.170:26856, ef9748625b4739c5411e276cf2cb0d2742a037f9@54.36.63.85:26656, 39490daffac0c7847b0d2617e412b2942055a82b@95.214.53.46:26656, 906114620df87a270b89404fdc7f15b3760fa34e@95.214.53.27:42656"

```

## Download address book

Besides adding seeds and peers you could download latest address book:
```bash
wget -O "$HOME/addrbook.json" https://snapshots1.polkachu.com/addrbook/gravity/addrbook.json
mv "$HOME/addrbook.json" ~/.gravity/config
```

### Configure your node for state sync

Follow [this guide](https://ping.pub/gravity-bridge/statesync) to configure your node for state sync if you do not skip this step you will have to start with Gravity v1.0 and upgrade when prompted. This will take a very long time.

### Start your full node and wait for it to sync

```bash
gravity start
```
