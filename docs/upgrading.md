# Gravity bridge Mercury Upgrade Part 2

As outlined in this [governance proposal](https://www.mintscan.io/gravity-bridge/proposals/40)

Gravity Bridge will be upgrading to Mercury part 2

* Fixes for the Ethereum -> Destination IBC forwarder + Extensive hand and automated testing

This upgrade *will not* change the chain-id and will occur at block height `1888600`

## Preparing for the upgrade

Since an upgrade proposal has passed no prep is required. Your node will automatically halt at the correct height.

This upgrade *does not* perform any store migrations, it contains only fixes so the risk of state corruption is significantly less than Mercury part 1

It is still *recommended* that you prepare enough disk space for a full backup of your node disk state. Or use a compact snapshot generated via [state sync](https://ping.pub/gravity-bridge/statesync) this will help you upgrade faster.

It is *required* that you backup your `_priv_validator_state.json`. If you do a full backup it will be included, but if you don't have the space backup at least this small file from `.gravity/data/priv_validator_state.json`. It will allow you to manually reconstruct a 'backup' from a snapshot and this file if required.

You may wish to research BTRFS and ZFS for your validator. As they will allow instant snapshots and backups that require no additional space on your disk.

## Backup your node

If your node has not yet halted at block `1888600` you are too early! Please wait.

Once the chain has reached block height `1888600` run `service gravity-bridge-stop`

**Once your node has halted it is recommended you backup your chain state**.

Make a complete copy of your `$HOME/.gravity` folder.

```bash
cp -r --reflink=auto ~/.gravity gravity-bridge-mercury-2-backup/
```

This will take a long time to run, if you are not using BTRFS or ZFS, you will need a lot of free disk space. If you do not have sufficient disk space to backup your entire gravity folder backup your `priv_validator_state.json`

```bash
cp ~/.gravity/data/priv_validator_state.json validator-state-backup.json
```

If all goes well you will not need either of these. If the upgrade fails we will roll back to this backup state and try to run it again, if you don't have a backup you will be unable to roll back, or worse you may double sign.

## Upgrading node software

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.2/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.2/gbt
chmod +x *
sudo mv * /usr/bin/
```

## **WARNING**

If you are using https for your orchestrator grpc endpoint you will need to modify your `/etc/systemd/system/orchestrator.service` file to use http instead. A tonic upgrade has broken http2/https support. Since most validators are using a their local validator node for GRPC this should not affect the vast majority of validators.

Run `service orchestrator status` after the upgrade, if you are experiencing this issue the big red error messages will be obvious.

## Restart the chain

```bash
service gravity-node start
service orchestrator restart
```

Your node will be slow to start as it performs the migrations, after that we will wait for all the other validators to join for the chain to resume.
