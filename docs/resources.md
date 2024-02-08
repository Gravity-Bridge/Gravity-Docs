# Resources

This document contains a list of block explorers and frontend interfaces for use with Gravity

## Bridge frontends

- [Blockscape](https://bridge.blockscape.network/#/)
- [Spacestation](https://spacestation.zone/)

## Governance Forum

Governance proposals should be coordinated using the [Commonwealth](https://commonwealth.im/gravity-bridge) forum.

## Blockexplorer

- [Mintscan Explorer](https://www.mintscan.io/gravity-bridge)
- [Aneka Explorer](https://gravity.aneka.io/)
- [Skynet Explorer](https://www.skynetexplorers.com/gravity-bridge)
- [Atomscan Explorer](https://atomscan.com/gravity-bridge)

## Gravity.sol Contract Address

The Gravity Bridge Blockchain contract address is.

[0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906](https://etherscan.io/address/0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906)

The Gravity Bridge contract ABI is available as `Gravity.json` attached to every [release](https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.8.1/Gravity.json)

## RPC

provides ABCI on `26657` Legacy RPC on port `1317` and gRPC on port `9090`

```text
https://gravitychain.io
```

## Testnet

The Gravity Bridge testnet is simply a copy of the [local developer environment](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/docs/developer/environment-setup.md) exposed to the outside world.

This environment is made up of two chains, one Gravity chain and another IBC test chain.

Testnet RPC

```text
http://testnet.gravitychain.io:1317  # Legacy RPC
http://testnet.gravitychain.io:9090  # gRPC
http://testnet.gravitychain.io:9091  # gRPC-web
http://testnet.gravitychain.io:26657 # ABCI


http://testnet.gravitychain.io:27657 # ABCI IBC network
http://testnet.gravitychain.io:9190  # GRPC IBC network

http://testnet.gravitychain.io:8545  # Ethereum
```

Keys containing test tokens

These is the private keys for the validators on the testnet

```text
solution capital echo near absent night fashion program expand kiss eager skirt right three victory urge trim patch aunt arm identify insane bottom auto

estate true cannon like screen joy hint slam vapor inner electric neither recycle hawk topple pond grow glad abandon confirm cupboard water chronic lounge

vital oppose scene vicious coach glue picnic absent tent rose ill source feel certain vicious patch jacket crew torch siege opinion artist ethics advice

kitchen security skin river olympic fee pluck amateur limb lava cover obtain isolate cabbage slim year ball night buyer replace push invite ozone initial

```

And the private keys for the validators on the second non-gravity IBC test network

```text
income suffer cruel soda capital whip urban remove margin possible eager grant detail almost result poet plug icon uniform brisk transfer dirt harvest tongue

clerk govern smart scene flavor tumble assist network acid tornado whip face remain transfer bridge disagree social spoon parade elite moment glimpse flee key

pet sea artefact portion state business marble project wing hurdle key sure notable payment similar famous finish tired main law inherit twenty deposit carbon

drink bleak town tunnel adjust orbit much lens clump decide differ clock review kiss outside police story minor soft proof orbit spoil ankle eye

```

This is the private key for the Ethereum miner, it has access to ETH and several ERC20's pre-deployed

```text
0xb1bab011e03a9862664706fc3bbaa1b16651528e5f0e7fbfcbfdd8be302a13e7
```

Test Gravity Bridge Address DO NOT SEND REAL FUNDS HERE, THIS IS A TESTNET ADDRESS

```text
Main Gravity 0x7580bFE88Dd3d07947908FAE12d95872a260F2D8
Gravity ERC721 0xD50c0953a99325d01cca655E57070F1be4983b6b
```

ERC20 tokens to test with, the miner private key has access
Also note that this testnet instance is using Hardhat, and thus has actual historical Ethereum state to interact with. It is possible to acquire some ETH from the Ethereum miner (see private key above) and swap for many ERC20s on Uniswap.

```text
0x0412C7c846bb6b7DC462CF6B453f76D8440b2609
0x30dA8589BFa1E509A319489E014d384b87815D89
0x9676519d99E390A180Ab1445d5d857E3f6869065
```

ERC721 NFTs to test with, the miner private key has access

```text
0xD7600ae27C99988A6CD360234062b540F88ECA43
```

## Chain history snapshots

### Recent snapshots

[Cros Nest (100/0/11)](https://quicksync.cros-nest.com/GravityBridge)
[Weakhand (default pruning)](https://snapshots.weakhand.xyz/gravity/)

### gravity-bridge-1

[google drive](https://drive.google.com/file/d/1LsGK-eBSfAditKHAJDBVQ0RhRRE4e8u0/view?usp=sharing)
torrent: WIP

### gravity-bridge-2

[google drive](https://drive.google.com/file/d/1gdYvEiLTDEGz1IY0kzhTG0ah5bz26uWW/view?usp=sharing)
torrent: WIP

### gravity-bridge-3 quicksync snapshot

[google drive](https://drive.google.com/file/d/187Aw_SpGNk7wwXgrYck6vvq8GogVw3t2/view?usp=sharing)
torrent: WIP
