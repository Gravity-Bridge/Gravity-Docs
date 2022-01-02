# Gravity bridge 2 upgrade

As outlined in this [governance proposal](https://github.com/Gravity-Bridge/Gravity-Docs/tree/main/gov/software-upgrade-v1_2.md)

We are upgrading from `gravity-bridge-1` to `gravity-bridge-2` in order to resolve a bug involving Gravity slashing and integrate several other bugfixes and improvements

## Preparing for the upgrade

In order to prepare for the upgrade stop your node and start it again with the argument

```text
--halt-height 309528
```

If you have followed the guide for [setting up your validator](/docs/setting-up-a-validator.md) then the file you need to edit is `/etc/systemd/system/gravity-node.service`

```text
ExecStart=/usr/bin/gravity start --halt-height 309528
```

Once you have completed this operation wait for the chain to reach block `309528` before following the below instructions. You can, if you desire, upgrade `gbt` now.

If you are using [Cosmosvisor](https://github.com/Gravity-Bridge/Gravity-Docs/blob/main/docs/upgrading.md) you may wish to set the environment flag `DAEMON_RESTART_AFTER_UPGRADE` to false so that Cosmosvisor does not attempt an in place upgrade.

## (Optional) Verifying The Upgraded Genesis

Stop! This step can not be done until the chain has halted on January 4th 2022 at block height `309528`.

```bash
# first stop your node to gain access to the database
service gravity-node stop

# now we will generate the new genesis.json
gravity export --height 309528 &> tmp.json
jq '' tmp.json > tmp-fmt.json
jq '.chain_id = "gravity-bridge-2"' tmp-fmt.json > gravity-bridge-2-genesis.json
md5sum gravity-bridge-2-genesis.json
```

You should see

```text
<upgrade not ready>  gravity-bridge-2-genesis.json
```

## Upgrading node software

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.2.0/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.2.0/gbt
chmod +x *
sudo mv * /usr/bin/
```

## Backup old chain state

Hopefully you won't need this, but if you do you'll be glad to have it. You may want to keep this archive around for future reference.

```bash
tar -czvf gravity-bridge-1.tar.gz ~/.gravity
```

## Restart the chain using the new genesis.json

Optionally confirm that this genesis.json has the same md5sum as the one you generated from your own state earlier

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
md5sum ~/.gravity/config/genesis.json
diff ~/.gravity/config/genesis.json gravity-bridge-2-genesis.json
```

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/genesis.json -O ~/.gravity/config/genesis.json
gravity unsafe-reset-all
systemctl daemon-reload
service gravity-node start
service orchestrator restart
```

Now we wait for the blockchain to successfully resume.
