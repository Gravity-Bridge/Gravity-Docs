# Gravity bridge Mercury Upgrade

As outlined in this [governance proposal](https://www.mintscan.io/gravity-bridge/proposals/27)

Gravity Bridge will be upgrading to Mercury, a version which includes several new features.

* IBC forwarding from Ethereum directly to a destination chain

* 5% minimum staking commission enforcement

* Improved indexing / store configuration

* Improved query endpoints for claims

* Improvements for Airdrops of Ethereum tokens

* Improved security checks for bridge operations

* Upgrade to CosmosSDK v0.44.6

This upgrade *will not* change the chain-id and will occur at block height `1282013`

## Preparing for the upgrade

Since an upgrade proposal has passed no prep is required. Your node will automatically halt at the correct height.

## Backup your node

If your node has not yet halted at block `1282013` you are too early! Please wait.

Once your node has halted **Once your node has halts you must backup your chain state**. Make a complete copy of your `$HOME/.gravity` folder.

```bash
tar -czvf gravity-bridge-pre-mercury.tar.gz ~/.gravity
```

This will take a long time to run, you will need a hundred GB or more free disk space. If you do not have sufficient disk space to backup your entire gravity folder backup your `priv_validator_state.json`

```bash
cp ~/.gravity/data/priv_validator_state.json validator-state-backup.json
```

If all goes well you will not need either of these. If the upgrade fails we will roll back to this backup state and try to run it again, if you don't have a backup you will be unable to roll back, or worse you may double sign.

## Double check your backups

Read the previous section again, make sure you followed the steps correctly. You probably won't need these backups but if you do need them they will help keep you from getting slashed or being unable to rejoin the validator set.

## Upgrading node software

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.0/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.0/gbt
chmod +x *
sudo mv * /usr/bin/
```

## Restart the chain

```bash
service gravity-node start
service orchestrator restart
```

Your node will be slow to start as it performs the migrations, after that we will wait for all the other validators to join for the chain to resume.

A few tips

* In place upgrades have a different risk profile than genesis upgrades, a lot more care needs to be taken with your local state and the potential for double signing
* Once your node is setup and waiting for a block feel free to go to sleep or walk away for a while.
* If you get an unexpected error ask in the chat before restarting your node
