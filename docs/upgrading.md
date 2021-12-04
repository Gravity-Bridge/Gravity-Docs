# Gravity bridge test2 upgrade

This mornings chain halt causing bug has been [resolved](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/8601913ba778f40488313a1b6f99735e0769a9f5) in order to deploy this fix Gravity Bridge will be updated to `gravity-bridge-test3`.

## Upgrading the Genesis

For this upgrade we are taking the opportunity to change two parameters, one is setting
the signed blocks window to ten thousand blocks. Which should reduce accidental slashing.

Second is to set the bridge Ethereum address, which helps the orchestrator run without manually passing it as a parameter every time.

```bash
# first stop your node to gain access to the database
service gravity-node stop

# now we will generate the new genesis.json
gravity export --height 9999 &> tmp.json
jq '' tmp.json > tmp-fmt.json
jq '.chain_id = "gravity-bridge-test3"' tmp-fmt.json > e0.json
jq '.app_state.slashing.params.signed_blocks_window = "10000"' e0.json > e1.json
jq '.app_state.gravity.params.bridge_ethereum_address = "0xa17f0B21c70FaB270c68031A179e7bE61BE7E81e"' e1.json > gravity-chain-test3-genesis.json
md5sum gravity-chain-test3-genesis.json
```

You should see

```text
bdced91c6c61aaecdb0080dc9d811f07  gravity-chain-test3-genesis.json
```

## Upgrading node software

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.3/gravity--linux-amd64
mv gravity--linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.3/gbt
chmod +x *
sudo mv * /usr/bin/
```

## Backup old chain state

Hopefully you won't need this, but if you do you'll be glad to have it

```bash
tar -czvf gravity-chain-test1.tar.gz ~/.gravity
```

## Restart the chain using the new genesis.json

confirm that this genesis.json has the same md5sum as the one you generated from your own state earlier

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
md5sum ~/.gravity/config/genesis.json
diff ~/.gravity/config/genesis.json gravity-chain-test3-genesis.json
```

Do not run *unsafe-reset-all* more than once per chain upgrade. If you do you risk double signing.

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
gravity unsafe-reset-all
service gravity-node start
```

Now we wait for the blockchain to successfully resume.
