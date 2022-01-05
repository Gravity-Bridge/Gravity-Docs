# Setting up your validator

Please see the [validator system setup](validator-system-setup.md) for recommended machine specs

We suggest an open notepad or other document to keep track of the keys you will be generating.

This document is *not* for genesis validators. They should use [settings up your genesis validator](/docs/setting-up-your-genesis-validator.md)

You will need a key containing some Graviton token. If you wish to participate in the network but not stake Graviton see the guide for [setting up a Gravity Bridge Chain full node](setting-up-a-fullnode.md)

## Download Gravity chain and the Gravity tools

```bash

mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.2.1/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.2.0/gbt
chmod +x *
sudo mv * /usr/bin/

```

At specific points during the testnet you may be told to 'update your orchestrator' or 'update your gravity binary'. In order to do that you can simply repeat the above instructions and then restart the affected software.

to check what version of the tools you have run `gbt --version` the current latest version is `gbt 1.2.0`

## Download the genesis file

The genesis file represents the current state of the blockchain and allows your node to sync up
with the rest.

```bash
gravity init mymoniker --chain-id gravity-bridge-2
wget https://raw.githubusercontent.com/Gravity-Bridge/gravity-docs/main/genesis.json
cp genesis.json $HOME/.gravity/config/genesis.json

```

## Add seed node

Change the seed field in ~/.gravity/config/config.toml to contain the following:

```text

seeds = "2b089bfb4c7366efb402b48376a7209632380c9c@65.19.136.133:26656,63e662f5e048d4902c7c7126291cf1fc17687e3c@95.211.103.175:26656"

```

### Add your validator key

We need to import the validator key. This is the key containing Graviton tokens

```bash
gravity keys add <my validator key name> --recover <your seed phrase>
```

Or if your key is stored in a ledger device.

```bash
gravity keys add <my validator key name> --ledger
```

### Generate your Delegate keys

There are three keys involved in this process.

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

You can stop your testnet node at this point. If you wish to keep the testnet running you should save this step until
mainnet start as it will cause your orchestrator to act strangely if you're using the same machine.

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

## Monitoring your logs

These lines will allow you to watch the logs coming out of your Gravity full node and Orchestrator as if you where directly attached to the process rather than using systemd. Run each in a separate terminal

```bash
journalctl -u gravity-node.service -f --output cat
journalctl -u orchestrator.service -f --output cat
```

## Setting up an Ethereum node

You probably noticed that your orchestrator is currently very unhappy. In this step we will setup an Ethereum light client

We will be using Geth Ethereum light clients for this task. For production Gravity we suggest that you point your Orchestrator at a Geth light client and then configure your light client to peer with full nodes that you control. This provides higher reliability as light clients are very quick to start/stop and resync. Allowing you to for example rebuild an Ethereum full node without having days of Orchestrator downtime.

Geth full nodes do not serve light clients by default, light clients do not trust full nodes, but if there are no full nodes to request proofs from they can not operate. Therefore we are collecting the largest possible
list of Geth full nodes from our community that will serve light clients.

If you have more than 40gb of free storage, an SSD and extra memory/CPU power, please run a full node and share the node url. If you do not, please use the light client instructions

### Please only run one or the other of the below instructions, both will not work

### Light client instructions

```bash

wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.10.13-7a0c19f8.tar.gz
tar -xvf geth-linux-amd64-1.10.13-7a0c19f8.tar.gz
cd geth-linux-amd64-1.10.13-7a0c19f8
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/configs/geth-light-config.toml
./geth --syncmode "light" --http --config geth-light-config.toml

```

### Fullnode instructions

```bash

wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.10.13-7a0c19f8.tar.gz
tar -xvf geth-linux-amd64-1.10.13-7a0c19f8.tar.gz
cd geth-linux-amd64-1.10.13-7a0c19f8
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/configs/geth-full-config.toml
./geth --http --config geth-full-config.toml

```

You'll see a url in this format, please note your ip and share both this node url and your ip in chat to add to the light client nodes list

```bash
INFO [06-10|14:11:03.104] Started P2P networking self=enode://71b8bb569dad23b16822a249582501aef5ed51adf384f424a060aec4151b7b5c4d8a1503c7f3113ef69e24e1944640fc2b422764cf25dbf9db91f34e94bf4571@127.0.0.1:30303
```

Finally you'll need to wait for several hours until your node is synced. Do not worry your orchestrator will submit signatures to to the Gravity bridge chain during this time.

## Wait for it

you will need to wait for your Gravity Bridge node and Ethereum Node to fully sync before progressing in the instructions

## Send your validator setup transaction

```bash

gravity tx staking create-validator \
 --amount=<the amount of graviton you wish to stake>ugraviton \
 --pubkey=$(gravity tendermint show-validator) \
 --moniker="put your validator name here" \
 --chain-id=gravity-bridge-2 \
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

Your delegate Ethereum key will need some Gorli ETH, your delegate cosmos key (orchestrator key) will need some tokens as well.
