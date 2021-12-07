# Gravity bridge test4 upgrade

This mornings chain halt causing bug has been [resolved](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/294ddc5b65840562422d44b0fd90db7254f69ee4) in order to deploy this fix Gravity Bridge will be updated to `gravity-bridge-test4`.

## (Optional) Verifying The Upgraded Genesis

For this upgrade we are taking the opportunity to change two parameters, one is setting
the signed blocks window to ten thousand blocks. Which should reduce accidental slashing.

Second is to set the bridge Ethereum address, which helps the orchestrator run without manually passing it as a parameter every time.

```bash
# first stop your node to gain access to the database
service gravity-node stop

# now we will generate the new genesis.json
gravity export --height 14969 &> tmp.json
jq '' tmp.json > tmp-fmt.json
jq '.chain_id = "gravity-bridge-test4"' tmp-fmt.json > gravity-chain-test4-genesis.json
md5sum gravity-chain-test4-genesis.json
```

You should see

```text
b17cf3db31edda601b5fcb431f3b7052  gravity-chain-test4-genesis.json
```

## Upgrading node software

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.4/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.4/gbt
chmod +x *
sudo mv * /usr/bin/
```

## Backup old chain state

Hopefully you won't need this, but if you do you'll be glad to have it

```bash
tar -czvf gravity-chain-test3.tar.gz ~/.gravity
```

## Restart the chain using the new genesis.json

Optional confirm that this genesis.json has the same md5sum as the one you generated from your own state earlier

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
md5sum ~/.gravity/config/genesis.json
diff ~/.gravity/config/genesis.json gravity-chain-test4-genesis.json
```

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
gravity unsafe-reset-all
service gravity-node start
```

Now we wait for the blockchain to successfully resume.
