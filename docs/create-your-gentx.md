# Gravity Bridge Genesis Validator Setup

## System setup

While Gravity Bridge full nodes can be run on any platform it is recommended you run your validator on Linux. The most up to date version of any particular distribution is recommended.

You can see [validator system setup](validator-system-setup.md) for more setup instructions. But the only part that's important for creating your gentx is that if you have an existing `.gravity` folder in your home directory.
Delete it

### Download the Gravity bridge binary

If you already have a .gravity folder from a previous run of `gravity init` delete it before this step.

```bash
mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.7/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.7/gbt
chmod +x *
sudo mv * /usr/bin/

gravity init <your moniker here>
```

### Add your validator key

First we need to import the validator key. This is the key for the address you submitted on the forum.

```bash
# you will be prompted for your key phrase
gravity keys add <my validator key name> --recover
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
gbt keys set-ethereum-key --key Gravity Orchestrator Ethereum Private Key
gbt keys set-orchestrator-key --phrase "Gravity Orchestrator Cosmos Key"
```

### Create a GenTX

```bash
wget https://raw.githubusercontent.com/gravity-bridge/gravity-docs/main/genesis.json
mv genesis.json ~/.gravity/config/genesis.json
gravity gentx --moniker <your moniker> <my validator key name> 181818181818ugraviton <Gravity Orchestrator Ethereum Address> <Gravity Orchestrator Cosmos Address> --chain-id=gravity-bridge-1
```

### Submit your gentx

In order to submit your gentx open up a pull request to this repo placing your gentx into the 'gentxs' folder
our automated test will check it for validity and after that it will be merged.
