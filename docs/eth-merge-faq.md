# Ethereum Merge FAQ

As you probably already know Ethereum is [moving to proof of stake](https://ethereum.org/en/upgrades/merge/) in an event called 'the merge'. As part of this event the current Ethereum chain will change it's consensus mechanism from proof of work to proof of stake on the Ethereum 'becon chain' or now simply 'consensus layer'.

This document covers how Gravity Bridge will interact with and secure tokenss during the merge and specifically teh precautions the community has chosen to take in the face of low probability, but possible, failures around the merge.

## When is the merge

The merge will take place at a 'Terminal Total Difficulty' of `58750000000000000000000`. Meaning the merge is triggered when the total amount of mining power for the Ethereum 1 chain exceeds this value. Exactly when that will happen is not perfectly predictable and it could be anywhere between September 14th 2022 and September 15th 2022. Estimation websites [like this one](bordel.wtf) provide an up to date estimation based on current mining power and speed.

## What are the risks of the merge to Gravity Bridge

The 24 hour estimated time range for the merge is an estimate based on the assumption that miners continue to operate normally. Considering their incentives to cause potential chaos around the merge we can not discount potential attacks to either delay or accelerate the merge.

Furthermore the successful transition to becon chain control may not be smooth. Chain halts, re-organizations, and consensus failures are all possible.

None of these are likley, in all probability the merge will go smoothly, but Gravity Bridge must be prepared regardless.

## How is Gravity Bridge preparing for the merge

tl;dr **Transfers to and from Ethereum will be halted for at least 48 hours leading up to the merge, after the merge valdiators will resume the bridge once ETH2 is deemed stable, probably around 24 hours later**

As part of the [Gravity Bridge Pleiades Upgrade](https://www.mintscan.io/gravity-bridge/proposals/74) new oracle software is being deployed to halt the bridge once the total difficulty exceeds `58591842390178918929502` this value is estimated to be about 48 hours before the merge. But as previously discussed timing is variable, regardless of when the merge actually occurs the Gravity Bridge oracle will automatically shut down the flow of information from Ethereum to Gravity Bridge around that time.

What this will do is prevent any double spend attacks caused by forks or chain re-organizations. As a secondary saftey measure a governance proposal to halt the bridge, also targted for an estimated 48 hours before the merge, will be submitted shortly after the Pleiades upgrade.

Combined these actions will ensure that Gravity Bridge does not reflect any balance changes on Ethereum until after the merge when the validators have upgraded their Ethereum nodes and have a stable stream of information coming from ETH2.

## What will happen to any deposits or withdraws I made while the bridge is paused

Gravity Bridge is designed to be paused and resumed safely. If you send funds to Gravity while the bridge is paused your funds will show up in your balance within a few minutes of the bridge resuming post merge.

On the Gravity Bridge side it will be possible to put your funds into the batch pool using a `MsgSendToEth` but they will not be batched while the bridge is paused. This means you may choose to remove your funds from the pool and use them elsewhere in the IBC ecosystem, or leave them there to be bridged to Ethereum once the bridge resumes.

## How will the bridge be resumed

Gravity Bridge will resume once the governance proposal to re-enable the bridge passes and the validators connect their `gbt` nodes to an updated ETH2 Ethereum RPC. Once this has been done the oracle will resume operaiton and all information about what has occured on Ethereum during the pause will be relayed and reflected on the Gravit Bridge side.

In the case of catastrophic failure of the merge it will be up to on chain governance to decide to delay unpausing the bridge or to select which fork to resume the bridge on. This is ultimately decided by which RPC 2/3 of the voting power choses to use for their `gbt` process.

## Does my USDC.grav represent ETH2 USDC or ETH POW USDC

Post merge, USDC.grav (or any .grav tokens for this example) will represent tokens on both ETH2, ETH POW and any other forks of Ethereum that may appear at this time.

Due to the design of Gravity Bridge any withdraw transactions can be played back on each fork of Ethereum. Meaning an advanced user will be able to claim bridge assets on all forks by sending them to themselves with a normal Gravity Bridge withdraw.

We **strongly recomend against interacting with Ethereum forks post merge** unless you have in-depth blockchain technical knowlege. If you must be sure to initialize a new wallet for each fork and transfer forked assets on each chain to that new wallet. Failure to take this precaution opens you to unexpected cross-chain interactions between the forks, at least until they recieve individual chain-id values.

## I'm a validator, how do I make sure everything is ready

First keep a close eye on governance votes, as they will convey more info and will be required to disable and re-enable the bridge.

Make sure `gbt --version` returns `v1.7.0` and start syncing an ETH becon chain node as as lighthouse as quickly as possible. Endpoints like `eth.althea.net` will continue to function but if you are running your own node you will need to run both a execution layer client such as Geth and a consensus layer client such as Lighthouse in order to operate an RPC.

When merge time approaches you will see the following line from `service orchestrator status`

```
[2022-09-04T21:38:55Z INFO  orchestrator::main_loop] Orchestrator has detected ETH2 merge will occur within 48 hours oracle is now halted
[2022-09-04T21:38:55Z INFO  orchestrator::main_loop] Current TDD 58591842390178918929502 ETH2 Merge 58750000000000000000000 Estimated 48.00 hours until the merge
```

This will count down leading up to the merge until the merge occurs, at which point you will see this line.

```
[2022-09-04T21:38:55Z INFO  orchestrator::main_loop] ETH2 Merge Complete! Please see Discord and Chain Governance for next steps to re-enable the bridge post merge!
```

Once the merge has completed a new release of `gbt` will become available, when connected to an updated ETH2 RPC your `gbt` process will send the required messages to resume the bridge.

