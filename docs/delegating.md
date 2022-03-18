# Delegating

This document covers how to delegate your Graviton token

## Delegating via Cosmostation

[Community guide for Cosmostation delegation (Without Ledger)](https://stakeking.notion.site/Gravity-Bridge-Delegation-Guide-c9c0ff7b190b45b9a4e6b7a54c62a996)

## Delegating via the command Line

### Linux (Ledger)

If you are running Linux and have your keys stored in a ledger do the following

```bash
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.0/gravity-linux-amd64
chmod +x gravity-linux-amd64
mv gravity-linux-amd gravity
sudo mv gravity /usr/bin/

gravity init gravity-bridge-3

gravity keys add --ledger <your ledger key name>

# you should see your ledger address
gravity keys list

gravity tx staking delegate --from <your ledger key name> <valoper address> <amount> --node https://gravitychain.io:26657 --chain-id gravity-bridge-3
```

### Mac (Ledger)

If you are running Mac and have your keys stored in a ledger do the following

```bash
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.0/gravity-darwin-amd64
sudo mv gravity-darwin-amd64 /usr/bin/

gravity init gravity-bridge-3

gravity keys add --ledger <your ledger key name>

# you should see your ledger address
gravity keys list

gravity tx staking delegate --from <your ledger key name> <valoper address> <amount> --node https://gravitychain.io:26657 --chain-id gravity-bridge-3
```

## Delegating via a web interface

This is coming soon
