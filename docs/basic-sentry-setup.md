# Setting Up a Sentry Node

## Background

Sentry nodes are vital to chain security, uptime and keeping your validator safe against DDOS attacks.  When running a validator on any cosmos sdk DPOS blockchain, node operators should consider implementing a sentry node architecture.  This may include any number of sentry nodes (full nodes) which relay messages between your validator and the rest of the validator set. Below are some basic instructions on setting up a sentry node. Note that a sentry node is essentially a full node with some modifications to the config file so the instructions for getting started are the same as the setting up a full node instructions in this repo.

## Setting up your full-node

## What do I need

A Linux server with any modern Linux distribution, 2gb of ram and at least 20gb storage.

### Download Gravity chain software

```bash
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.3.1/gravity-linux-amd64
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

### Add seed node

Change the seed field in ~/.gravity/config/config.toml to contain the following:

```text

seeds = "2b089bfb4c7366efb402b48376a7209632380c9c@65.19.136.133:26656,63e662f5e048d4902c7c7126291cf1fc17687e3c@95.211.103.175:26656"

```

### Start your full node and wait for it to sync

```bash
gravity start
```

### Modifying the config file

**Please only continue with this step once your sentry has synced.**

In order to properly configure both your validator and sentry you will need to know the node ID's.  In order to find the node ID use the command `gravity tendermint show-node-id`

#### Modify your SENTRY config.toml

```bash
cd ~/.gravity/config
nano config.toml
```

Modify the following lines in your config.toml and save

```text
- config.toml
    - [p2p] laddr = "tcp://<private_ip_address>:26656"
    - [p2p] external_address = "tcp://<public_ip_address>:26656"
    - seeds = "2b089bfb4c7366efb402b48376a7209632380c9c@65.19.136.133:26656,63e662f5e048d4902c7c7126291cf1fc17687e3c@95.211.103.175:26656"
    - persistent_peers = "9aa7e0c250466a2a147f8b7d2b886b0d45d44ca0@45.138.49.222:26656"
    - addr_book_strict = false
    - private_peer_ids = "<validator_node_ID@ip_address>:26656"
    - [fastsync] version = "v2" (optional)
```

#### Modify your VALIDATOR config.toml

In this example we are using 3 seperate sentries but you may use any number that you prefer. Please modify the config.toml on your Validator using the example below.

```text
  - config.toml
    - persistent_peers = "sentry-node-1-node-id@private-IP:26656,sentry-node-2-node-id@private-IP:26656,sentry-node-3-node-id@private-IP:26656"
    - addr_book_strict = false
    - max_num_inbound_peers = 0
    - pex = false
    - [fastsync] version = "v2" (optional)
```

Once you have modified the config.toml on your sentries and validator please restart your sentries and then your validator
