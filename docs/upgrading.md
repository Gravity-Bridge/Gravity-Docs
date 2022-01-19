# Gravity bridge 3 upgrade

As outlined in this [governance proposal](https://www.mintscan.io/gravity-bridge/proposals/6)

We are upgrading from `gravity-bridge-2` to `gravity-bridge-3` in order to provide a fix to those affected by the staking and delegation bug and deploy improvements to the IBCMetadata and Airdrop custom governance proposals. These upgrades will allow the Gravity Bridge community to move forward with more advanced usage of the bridge.

## Preparing for the upgrade

Since an upgrade proposal has passed no prep is required. Your node will automatically halt at the correct height.

Once your node has halted we will need to wait while the final slashing fixes are applied to the genesis file.

## (Optional) Verifying The Upgraded Genesis

```bash
# first stop your node to gain access to the database
service gravity-node stop

# now we will generate the new genesis.json
gravity export --height 487539 &> tmp.json
jq '' tmp.json > tmp-fmt.json
jq '.chain_id = "gravity-bridge-3"' tmp-fmt.json > gravity-bridge-3-genesis.json
md5sum gravity-bridge-3-genesis.json
```

You should see

```text
42c7c1cee8d912ef671af1aad89e7de6  gravity-bridge-3-genesis.json
```

This file *will differ* from the published genesis.json. You should perform a diff between your local state and the published state and confirm that the changes are the same as outlined [here](https://github.com/Gravity-Bridge/Gravity-Bridge/pull/20)

## Upgrading node software

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.3.3/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.3.3/gbt
chmod +x *
sudo mv * /usr/bin/
```

## Backup old chain state

Hopefully you won't need this, but if you do you'll be glad to have it. You may want to keep this archive around for future reference.

```bash
tar -czvf gravity-bridge-2.tar.gz ~/.gravity
```

## Remove skipping invariants

To prepare for the last upgrade you added the launch argument to skip invariants. That can now be removed since the chain state has been corrected.

If you have followed the guide for [setting up your validator](/docs/setting-up-a-validator.md) then the file you need to edit is `/etc/systemd/system/gravity-node.service` and remove the argument `--x-crisis-skip-assert-invariants`

```text
ExecStart=/usr/bin/gravity start
```

## Restart the chain using the new genesis.json

Optionally review the changes to the genesis.json made in [this pr](https://github.com/Gravity-Bridge/Gravity-Docs/pull/306). By diffing your local copy with the changed version.

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
md5sum ~/.gravity/config/genesis.json
diff ~/.gravity/config/genesis.json gravity-bridge-3-genesis.json
```

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
gravity unsafe-reset-all
systemctl daemon-reload
service gravity-node start
service orchestrator restart
```

Now we wait for the blockchain to successfully resume.
