# Gravity bridge Pleiades Upgrade - Phase I

As outlined in this [governance proposal](https://www.mintscan.io/gravity-bridge/proposals/74)

Gravity Bridge will be upgrading to Pleiades Phase I with the following features

* Updates to prepare for the Ethereum Merge, specifcially to protect Gravity Bridge from any forks or other merge problems. Please read the [ETH merge FAQ](/docs/eth-merge-faq.md)

* Improved integrity checks around batch timeouts, allowing for more aggressive timeout periods to be set by future governnace votes

* Corrected index hashing for BatchSendToEthClaim objects, the index now includes the Ethereum block height, completing the upgrade process for the vulnerability hot-patched in `v1.6.7`

This upgrade *will not* change the chain-id and will occur at block height `3608063`

## Preparing for the upgrade

Since an upgrade proposal has passed no prep is required. Your node will automatically halt at the correct height.

This upgrade will perform store migrations, as such you should perform a full backup of your node storage state as part of the upgrade procedure. If your validator setup does not allow rapid backups you should strongly consider performing a [state sync](https://ping.pub/gravity-bridge/statesync) this will help you upgrade faster by reducing the amount of data you have to back up.

It is ***required*** that you backup your `priv_validator_state.json`. If you do a full backup it will be included, but if you don't have the space backup at least this small file from `.gravity/data/priv_validator_state.json`. It will allow you to manually reconstruct a 'backup' from a snapshot and this file if required.

You may wish to research BTRFS and ZFS for your validator, as they will allow instant snapshots and backups that require no additional space on your disk.

## Backup your node

If your node has not yet halted at block `3608063` you are too early! Please wait and use one of these upgrade time estimation links [Mintscan](https://www.mintscan.io/gravity-bridge/blocks/3608063)

Once the chain has reached block height `3608063` run `service gravity-bridge-stop`

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

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.7.1/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.7.1/gbt
chmod +x *
sudo mv * /usr/bin/
```

### If you are validating on Mac, Windows, or Linux ARM

If you are validating on any of these platforms you will need to build your own binary.

```
git clone https://github.com/Gravity-Bridge/Gravity-Bridge
cd Gravity-Bridge/module
git checkout v1.7.1
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
4:22PM INF Successfully moved a batch to a new key! eth-block-height=15476491 event-nonce=13113 new-claim-hash=19a30c4fab45aa7bf6eb86ce5d06c6b469ff75e68bc3180cfcdb43e5191beac0 new-key=0bfa165ff4ef558b3d0b62ea4d4a46c5000000000000333919a30c4fab45aa7bf6eb86ce5d06c6b469ff75e68bc3180cfcdb43e5191beac0 old-claim-hash=d3078163b6a4522933ac64b194cc4335b0bcb952bc12c260ed8e9661bfb1cf5b old-key=0bfa165ff4ef558b3d0b62ea4d4a46c50000000000003339d3078163b6a4522933ac64b194cc4335b0bcb952bc12c260ed8e9661bfb1cf5b type=CLAIM_TYPE_BATCH_SEND_TO_ETH
4:22PM INF Successfully moved a batch to a new key! eth-block-height=15476511 event-nonce=13114 new-claim-hash=84bdc1802fdb30633fab7d181a7c88c8ad6294bfe58390fab3cd93586c186e8b new-key=0bfa165ff4ef558b3d0b62ea4d4a46c5000000000000333a84bdc1802fdb30633fab7d181a7c88c8ad6294bfe58390fab3cd93586c186e8b old-claim-hash=c0e70d66a58e184dfbac08739da2e356aca30dec3876bd19884e70dadada118f old-key=0bfa165ff4ef558b3d0b62ea4d4a46c5000000000000333ac0e70d66a58e184dfbac08739da2e356aca30dec3876bd19884e70dadada118f type=CLAIM_TYPE_BATCH_SEND_TO_ETH
4:22PM INF Successfully moved a batch to a new key! eth-block-height=15476703 event-nonce=13121 new-claim-hash=91e8b743a0f1dc3f270cacb0cdba118cec6b3f4ea1d5d05fc30144a38a38a29e new-key=0bfa165ff4ef558b3d0b62ea4d4a46c5000000000000334191e8b743a0f1dc3f270cacb0cdba118cec6b3f4ea1d5d05fc30144a38a38a29e old-claim-hash=7dde730ae5d8dada5961887594f91197e9644479d36ef2161b90c202e678aad2 old-key=0bfa165ff4ef558b3d0b62ea4d4a46c500000000000033417dde730ae5d8dada5961887594f91197e9644479d36ef2161b90c202e678aad2 type=CLAIM_TYPE_BATCH_SEND_TO_ETH
4:24PM INF asserting crisis invariants inv=2/12 module=x/crisis name=bank/total-supply
4:24PM INF asserting crisis invariants inv=3/12 module=x/crisis name=gov/module-account
4:24PM INF asserting crisis invariants inv=4/12 module=x/crisis name=distribution/nonnegative-outstanding
4:24PM INF asserting crisis invariants inv=5/12 module=x/crisis name=distribution/can-withdraw
4:26PM INF asserting crisis invariants inv=6/12 module=x/crisis name=distribution/reference-count
4:26PM INF asserting crisis invariants inv=7/12 module=x/crisis name=distribution/module-account
4:26PM INF asserting crisis invariants inv=8/12 module=x/crisis name=staking/module-accounts
4:26PM INF asserting crisis invariants inv=9/12 module=x/crisis name=staking/nonnegative-power
4:26PM INF asserting crisis invariants inv=10/12 module=x/crisis name=staking/positive-delegation
4:26PM INF asserting crisis invariants inv=11/12 module=x/crisis name=staking/delegator-shares
4:26PM INF asserting crisis invariants inv=12/12 module=x/crisis name=gravity/module-balance
4:26PM INF asserted all invariants duration=218591.271816 height=3595782 module=x/crisis

```

> **If you do not have logs like this** then you should reach out in the `#validators` discord channel.
