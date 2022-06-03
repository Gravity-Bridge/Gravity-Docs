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

The Gravity Bridge contract ABI is available as `Gravity.json` attached to every [release](https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.2/Gravity.json)

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
safe useless favorite lemon wool mandate ladder army tuna quit aware drift unfair chicken absorb evidence balcony strike wolf stay kid amused head august

garden trip dawn gas mosquito attack negative vote finger upon ice buffalo guitar layer canoe high bubble tank admit curve raw shiver cook antique

pause eye idea blanket wage language believe distance party admit version twenty congress false machine express actress hen wing solar wrap scan hover beef

deputy tiny pupil live wash recycle erosion text physical hawk damp message curve blush theme eye warm message hurt yard embark evil tower stove
```

And the private keys for the validators on the second non-gravity IBC test network

```text
guilt harsh risk code guard boat snake omit borrow legal delay view tonight shoot opinion remember slide trust beyond steak card atom again bamboo

section fatal position attack nurse clip above across suggest style benefit hawk surprise scan pulse shove blame cherry light sweet tumble select shop oppose

off abuse alarm hint fat aerobic youth apple spring trap soccer sunset powder unable anger learn immune valve salmon disease festival sting symptom hawk

artist moon mutual luxury truly portion follow maid enter umbrella upgrade circle hole concert blouse humble insect sketch dinner atom fatal genius conduct drink
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
