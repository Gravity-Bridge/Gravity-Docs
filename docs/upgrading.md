# Gravity bridge test5 upgrade

We are upgrading from `gravity-bridge-test4` to `gravity-bridge-test5` in order to test new software and bugfixes. You can find details in the commit history
on the Gravity Bridge repo.

## (Optional) Verifying The Upgraded Genesis

```bash
# first stop your node to gain access to the database
service gravity-node stop

# now we will generate the new genesis.json
gravity export --height 59000 &> tmp.json
jq '' tmp.json > tmp-fmt.json
jq '.chain_id = "gravity-bridge-test5"' tmp-fmt.json > gravity-chain-test5-genesis.json
md5sum gravity-chain-test5-genesis.json
```

You should see

```text
a28da66975b855d07312f15d312b1822  gravity-chain-test5-genesis.json
```

## Upgrading node software

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.6/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.6/gbt
chmod +x *
sudo mv * /usr/bin/
```

## Backup old chain state

Hopefully you won't need this, but if you do you'll be glad to have it

```bash
tar -czvf gravity-chain-test4.tar.gz ~/.gravity
```

## Restart the chain using the new genesis.json

Optional confirm that this genesis.json has the same md5sum as the one you generated from your own state earlier

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
md5sum ~/.gravity/config/genesis.json
diff ~/.gravity/config/genesis.json gravity-chain-test5-genesis.json
```

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
gravity unsafe-reset-all
service gravity-node start
```

Now we wait for the blockchain to successfully resume.
