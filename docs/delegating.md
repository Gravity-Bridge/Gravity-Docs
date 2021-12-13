# Delegating

This file covers how to delegate your Graviton token

## Delegating via the command Line

### Linux (Ledger)

If you are running Linux and have your keys stored in a ledger do the following

```bash 
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.8/gravity-linux-amd64
sudo mv gravity-linux-amd64 /bin/

gravity init gravity-bridge-1

gravity keys add --ledger <your ledger key name>

# you should see your ledger address
gravity keys list 

gravity tx staking delegate --from <your ledger key name> <valoper address> --node http://chainripper-2.althea.net:26657 --chain-id gravity-bridge-1
```

### Mac (Ledger)

If you are running Linux and have your keys stored in a ledger do the following

```bash 
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.8/gravity-darwin-amd64
sudo mv gravity-darwin-amd64 /bin/

gravity init gravity-bridge-1

gravity keys add --ledger <your ledger key name>

# you should see your ledger address
gravity keys list 

gravity tx staking delegate --from <your ledger key name> <valoper address> --node http://chainripper-2.althea.net:26657 --chain-id gravity-bridge-1
```

## Delegating via a web interface

This is coming soon