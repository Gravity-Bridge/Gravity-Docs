# Gravity Genesis Bridge Validator Setup

This guide requires that you have already submitted and had merged a gentx from [these instructions](create-your-gentx.md)

If you have not done so you will not be able to start validating as a genesis validator and you will have to join using the post-launch [setting up a validator](setting-up-a-validator.md) instructions

If you are validating on a different machine than the one that signed your gentx, make sure to copy over your `.gravity` folder!

Do *not* set a min-fee for your validator until after the chain has started. On chain start you will have no liquid `ugraviton` that can be used to pay fees. Since your orchestrator will need to submit Ethereum signatures to prevent you from getting slashed you must allow zero fee transactions until liquid `ugraviton` is available to fund your orchestrator.

## Download Gravity tools

```bash

mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.0/gravity-v0.1.5-linux-amd64
mv gravity-v0.1.5-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.0/gbt
chmod +x *
sudo mv * /usr/bin/

```

At specific points during the testnet you may be told to 'update your orchestrator' or 'update your gravity binary'. In order to do that you can simply repeat the above instructions and then restart the affected software.

to check what version of the tools you have run `gbt --version` the current latest version is `gbt 0.5.6`

## Download updated Genesis.json

This genesis.json contains all gentx's that everyone has submitted, together they will create the first block of a new chain.

```bash
wget https://raw.githubusercontent.com/Gravity-Bridge/gravity-docs/main/genesis.json genesis.json
cp genesis.json $HOME/.gravity/config/genesis.json
```

## Setup Gravity Bridge Tools

```bash
gbt init
gbt keys set-ethereum-key --key <your Ethereum private key generated during gentx creation>
gbt keys set-orchestrator-key --phrase "the orchestrator key phrase you generated with your gentx"
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
--fees <fees> \
```

Now that we have modified these services it's time to set them to run on startup

```bash
sudo systemctl daemon-reload
sudo systemctl enable gravity-node
sudo systemctl enable orchestrator
sudo service gravity-node start
sudo service orchestrator start
```

Once you have completed this setup your node will be started and waiting for the chain to move in the background.

## Monitoring your logs

These lines will allow you to watch the logs coming out of your Gravity full node and Orchestrator as if you where directly attached to the process rather than using systemd. Run each in a separate terminal

```bash
journalctl -u gravity-chain.service -f --output cat
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
./geth --syncmode "light" --goerli --http --config geth-light-config.toml

```

### Fullnode instructions

```bash

wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.10.13-7a0c19f8.tar.gz
tar -xvf geth-linux-amd64-1.10.13-7a0c19f8.tar.gz
cd geth-linux-amd64-1.10.13-7a0c19f8
wget https://raw.githubusercontent.com/Gravity-Bridge/Gravity-Docs/main/configs/geth-full-config.toml
./geth --goerli --http --config geth-full-config.toml

```

You'll see a url in this format, please note your ip and share both this node url and your ip in chat to add to the light client nodes list

```bash
INFO [06-10|14:11:03.104] Started P2P networking self=enode://71b8bb569dad23b16822a249582501aef5ed51adf384f424a060aec4151b7b5c4d8a1503c7f3113ef69e24e1944640fc2b422764cf25dbf9db91f34e94bf4571@127.0.0.1:30303
```

Finally you'll need to wait for several hours until your node is synced. Do not worry your orchestrator will submit signatures to to the Gravity bridge chain during this time.

## Wait for it

Now that you have your Gravity node, Orchestrator, and Ethereum node setup it's time to settle in and wait for the chain to start. Once it does the Gravity bridge contract will be deployed and the validators will have to adopt it by changing orchestrator configs and then by governance vote.

Your delegate Ethereum key will need Gorli Eth