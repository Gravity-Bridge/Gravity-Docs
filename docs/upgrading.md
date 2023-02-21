# Gravity bridge Pleiades Upgrade - Phase II

As outlined in this [governance proposal](https://www.mintscan.io/gravity-bridge/proposals/105)

Gravity Bridge will be upgrading to Pleiades Phase II with the following features

* Automated invariants, by default validators will run chain invariants every 17 to 199 blocks

* Full store check invariant has been added. This cross references all Gravity store data and will halt the chain if anything becomes inconsistent, preventing attacks around store corruption.

* Improved Finality efficiency, deposits to GB will re recognized ~7 minutes faster thanks to improved ETH2 finality detection

* Fees, this update will implement Gravity Bridge fees as outlined in [proposal-86](https://www.mintscan.io/gravity-bridge/proposals/86), with a parameter controlling minimum feed editable by governance

This upgrade *will not* change the chain-id and will occur at block height `5264000`

## Preparing for the upgrade

Since an upgrade proposal has passed no prep is required. Your node will automatically halt at the correct height.

This upgrade will perform store migrations, as such you should perform a full backup of your node storage state as part of the upgrade procedure. If your validator setup does not allow rapid backups you should strongly consider performing a [state sync](https://ping.pub/gravity-bridge/statesync) this will help you upgrade faster by reducing the amount of data you have to back up.

It is ***required*** that you backup your `priv_validator_state.json`. If you do a full backup it will be included, but if you don't have the space backup at least this small file from `.gravity/data/priv_validator_state.json`. It will allow you to manually reconstruct a 'backup' from a snapshot and this file if required.

You may wish to research BTRFS and ZFS for your validator, as they will allow instant snapshots and backups that require no additional space on your disk.

## Backup your node

If your node has not yet halted at block `5264000` you are too early! Please wait and use one of these upgrade time estimation links [Mintscan](https://www.mintscan.io/gravity-bridge/blocks/5264000)

Once the chain has reached block height `5264000` run `service gravity-bridge-stop`

**Once your node has halted it is recommended you backup your chain state**.

Make a complete copy of your `$HOME/.gravity` folder. Exactly how you make this copy is up to you and depending on your system setup some methods may be dramatically faster than others.

If you are using a copy-on-write filesystem like BTRFS or ZFS a simple copy will be the fastest possible backup method.

```bash
cp -r --reflink=always ~/.gravity gravity-bridge-pleiades-backup/
```

If you have `pigz` available for parallel gzip compression you can use

```bash
tar --use-compress-program=pigz -cvf gravity-bridge-pleiades-backup.tar.gz ~/.gravity
```

If you have LVM snapshots available you may use them as well.

Finally if none of these quick backup options are available to you you should follow [these instructions](https://ping.pub/gravity-bridge/statesync) to statesync your node, dramatically reducing the size of the Gravity folder in exchange for not having historical block data.

Once you have state synced you can quickly and easily backup with just the cp command.

```bash
cp -r ~/.gravity gravity-bridge-pleiades-backup/
```

If you do not have sufficient disk space to backup your entire gravity folder then backup your `priv_validator_state.json`

```bash
cp ~/.gravity/data/priv_validator_state.json validator-state-backup.json
```

If all goes well you will not need either of these. If the upgrade fails we will roll back to this backup state and try to run it again, if you don't have a backup you will be unable to roll back, or worse you may double sign.

## Upgrading node software

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.8.1/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.8.1/gbt
chmod +x *
sudo mv * /usr/bin/
```

### If you are validating on Mac, Windows, or Linux ARM

If you are validating on any of these platforms you will need to build your own binary.

```
git clone https://github.com/Gravity-Bridge/Gravity-Bridge
cd Gravity-Bridge/module
git checkout v1.8.1
make install
cp $GOPATH/bin/gravity /usr/bin/gravity
```

## **WARNING**

Run `service orchestrator status` after the upgrade, if you are experiencing any issues big red error messages will be obvious. If your orchestrator is not running correclty you may be slashed in about 10,000 blocks. So be sure to check

## Restart the chain

```bash
service gravity-node start
service orchestrator restart
```

Your node will be slow to start as it performs the migrations, after that we will wait for all the other validators to join for the chain to resume.

## Expected Upgrade Logs

Shortly after upgrading you should see some log output like the following when the pleiades upgrade runs. There will be many batch move messages:

```text
4:58PM INF Pleiades Upgrade part 2: Enter handler
4:58PM INF Pleiades Upgrade part 2: Running any configured module migrations
4:58PM INF migrating module gravity from version 3 to version 4
4:58PM INF Pleiades Upgrade part 2: Enter Migrate3to4()
4:58PM INF Pleiades Upgrade part 2: Beginning the migrations for the gravity module
4:58PM INF Pleiades Upgrade part 2: Finished the migrations for the gravity module successfully!
4:58PM INF Pleiades Upgrade part 2: Gravity module migration is complete!
4:58PM INF migrating module transfer from version 1 to version 2
4:58PM INF Pleiades Upgrade part 2: Enforcing validator minimum comission
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): Enter function
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): Getting all the validators
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): minCommissionRate=0.100000000000000000
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): Iterating validators
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): checking validator Commission.MaxRate=0.200000000000000000 Commission.Rate=0.100000000000000000 validator="🔥STAVR🔥"
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): checking validator Commission.MaxRate=0.200000000000000000 Commission.Rate=0.060000000000000000 validator=Terminet
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): validator is out of compilance! Modifying their commission rate(s) validator =Terminet
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): Updating validator Commission.Rate
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): also UpdateTime  new="2023-01-03 00:49:57.857858311 +0000 UTC" old="2022-10-25 07:17:14.728392008 +0000 UTC"
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): calling the hook
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): setting the validator
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): validator's set rate Commission="commissionrates:\n  rate: \"0.100000000000000000\"\n  max_rate: \"0.200000000000000000\"\n  max_change_rate: \"0.010000000000000000\"\nupdate_time: 2023-01-03T00:49:57.857858311Z\n" validator=Terminet
4:58PM INF Pleiades Upgrade part 2: bumpMinValidatorCommissions(): checking validator Commission.MaxRate=0.200000000000000000 Commission.Rate=0.100000000000000000 validator=emir
.... Many more lines of this ....
5:03PM INF asserting crisis invariants inv=9/13 module=x/crisis name=gov/module-account
5:03PM INF asserting crisis invariants inv=10/13 module=x/crisis name=staking/module-accounts
5:03PM INF asserting crisis invariants inv=11/13 module=x/crisis name=staking/nonnegative-power
5:03PM INF asserting crisis invariants inv=12/13 module=x/crisis name=staking/positive-delegation
5:03PM INF asserting crisis invariants inv=13/13 module=x/crisis name=staking/delegator-shares
5:03PM INF asserted all invariants duration=295849.235283 height=5241190 module=x/crisis

```

> **If you do not have logs like this** then you should reach out in the `#validators` discord channel.
