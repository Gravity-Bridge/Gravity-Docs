# Setting up your validator

Please see the [validator system setup](validator-system-setup.md) for recommended machine specs

We suggest an open notepad or other document to keep track of the keys you will be generating.

This document is *not* for genesis validators. They should use [settings up your genesis validator](/docs/setting-up-your-genesis-validator.md)

You will need a key containing some Graviton token. If you wish to participate in the network but not stake Graviton see the guide for [setting up a Gravity Bridge Chain full node](setting-up-a-fullnode.md)

Once your validator is setup consider [setting up a sentry node](basic-sentry-setup.md)

## Download Gravity chain and the Gravity tools

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

At specific points you may be told to 'update your orchestrator' or 'update your gravity binary'. In order to do that you can simply repeat the above instructions and then restart the affected software.

to check what version of the tools you have run `gbt --version` the current latest version is `gbt 1.5.0`

## Download and install geth

You only need to do this if you are running Geth locally

```bash
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.10.15-8be800ff.tar.gz
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/configs/geth-light-config.toml -O /etc/geth-light-config.toml
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/configs/geth-full-config.toml -O /etc/geth-full-config.toml
tar -xvf geth-linux-amd64-1.10.15-8be800ff.tar.gz
cd geth-linux-amd64-1.10.15-8be800ff
mv geth /usr/sbin/
```

## Download the genesis file

The genesis file represents the current state of the blockchain and allows your node to sync up
with the rest.

```bash
gravity init mymoniker --chain-id gravity-bridge-3
wget https://raw.githubusercontent.com/Gravity-Bridge/gravity-docs/main/genesis.json
cp genesis.json $HOME/.gravity/config/genesis.json

```

## Add seed node

Change the seed field in ~/.gravity/config/config.toml to contain the following:

```text

seeds = "2b089bfb4c7366efb402b48376a7209632380c9c@65.19.136.133:26656,63e662f5e048d4902c7c7126291cf1fc17687e3c@95.211.103.175:26656"

```

### Configure your node for state sync OR Download a snapshot

Note that the most secure way to sync the chain is to sync the data yourself instead of using a snapshot or statesync. But it takes a long time and several software version upgrades.

If you are syncing from scratch you will need to start with Gravity Bridge version v1.0.0 and upgrade when prompted.

Follow [this guide](https://ping.pub/gravity-bridge/statesync) to configure your node for state sync. Or follow the steps below to download a snapshot.

```text
https://cosmos-snapshots.s3.filebase.com/gravitybridge/snapshot.json
```

(This is from the project <https://github.com/ovrclk/cosmos-omnibus/tree/master/gravitybridge> ) </br>
In the snapshot.json look at the line 'latest', it should look something like:

```text
"latest": "https://cosmos-snapshots.s3.filebase.com/gravitybridge/gravity-bridge-<date>-<time>.tar.gz"
```

Download and unzip that that file, use the contents to replace your gravity data folder which should be located at:

```bash
.gravity/data/
```

### Add your validator key

We need to import the validator key. This is the key containing Graviton tokens

```bash
# you will be prompted for your key phrase
gravity keys add <my validator key name> --recover
```

Or if your key is stored in a ledger device.

```bash
gravity keys add <my validator key name> --ledger
```

#### View your key (optional)

If you need to view the address of your validator operator key, you can do so with the following command: </br>

```bash
gravity keys show <validator key name> --bech val
```

You should see an output like so:

```text
- name: <validator key name>
  ...
  address: gravityvaloper<keystring>
  ...
```

### Generate your Delegate keys

There are four keys involved in this process.

```text
Validator Funds key: This is the key you submitted for genesis it starts with `gravity1` and contains your funds

Validator Operator Key: This is a key that will be generated with your gentx, it starts with `gravityvaloper1` and actually signs your validators blocks

Gravity Orchestrator Cosmos Key: This is a key that will be used on the Cosmos side of Gravity bridge to submit Oracle transactions and Ethereum signatures. This address will be actively used by Gravity bridge to send many hundreds of messages during normal day to day operation of an active bridge. You will be generating this key to register as part of your gentx.

Gravity Orchestrator Ethereum Key: This is an Ethereum key this is the key that represents your validators voting power on Ethereum in the `Gravity.sol` contract. In short this key secures the Gravity Bridge funds on Ethereum. This key will *not* be actively used to submit messages to Ethereum unless you chose to relay in addition to validate. Like the Gravity Orchestrator Cosmos Key you will be generating this key here and registering it as part of your gentx.

```

Together we may refer to the Gravity Orchestrator Cosmos Key and Gravity Orchestrator Ethereum Keys as 'Gravity delegate keys' as they act as a 'delegate' for your Validator Operator key.

If you lose your Gravity delegate keys you will have to unbond and create a new validator because it's not possible to rotate them. So store all output of the following commands in a safe place.

```bash
gravity eth_keys add
gravity keys add <Your Gravity Orchestrator Cosmos Key Name>
```

Once we have registered our keys we will also set them in our Orchestrator right away, this reduces the risk of confusion as the chain starts and you need these keys to submit Gravity bridge signatures via your orchestrator.

```bash
gbt init
gbt keys set-ethereum-key --key Gravity Orchestrator Ethereum Key
gbt keys set-orchestrator-key --phrase "Gravity Orchestrator Cosmos Key"
```

## Setup Gravity Bridge and Orchestrator services

We recommend using systemd to manage your validator and orchestrator processes.

systemd makes it very easy to increase the open files limit for validators and ensure auto restart on failure.
You can run `gravity start` without systemd, but if you do so be absolutely sure you have increased the system
open files limit.

```bash
cd /etc/systemd/system
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/configs/gravity-node.service
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/configs/orchestrator.service
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/configs/geth.service
```

Now we have to stop and customize these services as appropriate

If you are running your validator as the `root` user, the `gravity-node.service` file requires no modification
if you are running as a different user modify line 13 like so

```text
Environment="HOME=/path/to/your/home/dir"
```

For the orchestrator most people will not need to modify anything. But if you wish to specify an Eth node on a different machine or any other set of arguments modify line 10 as follows. As noted earlier in the instructions you should not set a minimum fee now, but once you do you'll have to return and set a minimum fee for your orchestrator here.

```text
ExecStart=/usr/bin/gbt orchestrator \
--ethereum-rpc <ETHEREUM_RPC> \
--fees <fees>
```

For the Geth node, if you are going to run a geth full node delete lines 11-15 and uncomment lines 17-21

## Run Validator Services

Now that we have modified these services it's time to set them to run on startup

```bash
sudo systemctl daemon-reload
sudo systemctl enable gravity-node
sudo systemctl enable orchestrator
sudo systemctl enable geth
sudo service gravity-node start
sudo service orchestrator start
sudo service geth start
```

Once you have completed this setup your node will be started and waiting for the chain to move in the background.

## Troubleshooting SELinux

If your services are not starting, you may want to try disabling it to see if it resolves the issue </br>

```bash
sed -i""  -e "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
```

```bash
setenforce 0
```

## Monitoring your logs

These lines will allow you to watch the logs coming out of your Gravity full node and Orchestrator as if you where directly attached to the process rather than using systemd. Run each in a separate terminal

Expect to see errors in your Orchestrator service, this will persist until we finish setting up the Orchestrator in the next few steps. Since finishing Orchestrator setup requires a synced Gravity and Ethereum node there's nothing to do about it now.

```bash
journalctl -u gravity-node.service -f --output cat
journalctl -u orchestrator.service -f --output cat
journalctl -u geth.service -f --output cat
```

## Observe Sync Status and Time Remaining

You will need to wait for your Gravity Bridge node and Ethereum Node to fully sync before progressing in the instructions </br>

### Ethereum

You can view the status of your Ethereum node by issuing the following command: </br>

```bash
curl -H "Content-Type:application/json" -X POST -d '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://127.0.0.1:8545
```

When result is 'false' that means it is now synced ('true' is not synced).

### Gravity Node

You can issue the following command to check the sync status of the Gravity Node </br>

```bash
gravity status 2>&1| jq .SyncInfo.catching_up
```

Value of 'false' means that it is now synced, 'true' means that sync is still in process.

### Look at Time Remaining for Gravity Sync

If you look at your journal logs for gravity-node like so:

```bash
journalctl -u gravity-node.service -f --output cat
```

You should see a message in the logs that looks like (search for 'fast'):

```text
9:49PM INF committed state app_hash=76B7FFFA844FA8EABA6E2C400DBE53C22A6F94A36E41922F06C0D57417E118EB height=350595 module=state num_txs=1
9:49PM INF Fast Sync Rate blocks/s=6.0707074013746825 height=350596 max_peer_height=422352 module=blockchain
9:49PM INF indexed block height=350595 module=txindex
```

You can calculate remaining time in seconds with:

```text
(max_peer_height - height) / Fast Sync Rate blocks/s
```

## Send your validator setup transaction

```bash

gravity tx staking create-validator \
 --amount=<the amount of graviton you wish to stake>ugraviton \
 --pubkey=$(gravity tendermint show-validator) \
 --moniker="put your validator name here" \
 --chain-id=gravity-bridge-3 \
 --from=myvalidatorkeyname \
 --commission-rate="0.10" \
 --commission-max-rate="0.20" \
 --commission-max-change-rate="0.01" \
 --gas=auto \
 --min-self-delegation="1" \
 --gas-adjustment=1.4


```

## Confirm that you are validating

If you see one line in the response you are validating. If you don't see any output from this command you are not validating. Check that the last command ran successfully.

Be sure to replace 'my validator key name' with your actual key name. If you want to double check you can see all your keys with 'althea keys list'

```bash

gravity query staking validator $(gravity keys show myvalidatorkeyname --bech val --address)

```

## Setup Gravity bridge

You are now validating on the Gravity Bridge blockchain. But as a validator you also need to run the Gravity bridge components or you will be slashed and removed from the validator set after about 16 hours.

### Register your delegate keys

Delegate keys allow the for the validator private keys to be kept in secure storage while the Orchestrator can use it's own delegated keys for Gravity functions. The delegate keys registration tool will generate Ethereum and Cosmos keys for you if you don't provide any. These will be saved in your local config for later use.

\*\*If you have set a minimum fee value in your `~/.gravity/config/app.toml` modify the `--fees` parameter to match that value!

```bash

gbt init

gbt keys register-orchestrator-address --validator-phrase "the phrase you saved earlier" --ethereum-key "the key you saved earlier" --fees=125000ugraviton

```

### Registering your delegate keys using a Ledger

**If you ran the above command skip this step** this is an **optional** step for those who are using hardware security for their validator key. In order to register your keys you will use the `gravity` cli instead of `gbt`. You will need to generate one ethereum key and one cosmos key yourself.

```bash

gravity tx gravity set-orchestrator-address [validator-address] [orchestrator-address] [ethereum-address]

```

### Fund your delegate keys

Your delegate Ethereum key will need some ETH (dust enough), your delegate cosmos key (orchestrator key) will need some tokens as well.
