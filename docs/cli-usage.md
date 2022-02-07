# Using Gravity Bridge via the CLI

This guide is for advanced users who want more control than a GUI will provide them or simply prefer advanced tools.

If you are looking for an easy to use GUI please see the [resources](resources.md) page.

## Download and install CLI tools

Because the release builder has not yet been updated to cross-compile to mac, `gbt` is currently Linux and Windows only.

### Linux

```bash

mkdir gravity-bin
cd gravity-bin

# the gravity chain binary itself

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.4.0/gravity-linux-amd64
mv gravity-linux-amd64 gravity

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.4.0/gbt
chmod +x *
sudo mv * /usr/bin/

```

### Windows

Download [gravity](https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.4.0/gravity-windows-amd64.exe) and [gbt](https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.4.0/gbt.exe)

You will need to open a terminal and, before running any commands, nagivate to the directory where these files were downloaded.

## Sending ERC20 tokens from Ethereum to Gravity Bridge

Once sent, tokens should arrive at the destination address on Gravity Bridge within 5 minutes. They will be named `gravity0xERC20ADDRESS`.

The send will cost about 70k gas on Ethereum.

Note: if you send tokens to a `cosmos1` address, you will find them in the same private key on Gravity Bridge. Which is to say, if you import the private key for your `cosmos1` address using 'gravity keys add', you will find your tokens there.

The easiest way to run the command is to paste the following in a text editor, modify as needed, and then copy/paste it into a terminal.

```bash
gbt client eth-to-cosmos --amount 10.5 \
--token-contact-address "0xERC20ADDRESS" \
--destination "gravity1EXAMPLEADDRESS" \
--ethereum-key "0xETHPRIVATEKEY" \
--gravity-contract-address "0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906" \
--ethereum-rpc https://eth.althea.net
```

View the tokens in your account

```bash
gravity query bank balances gravity1EXAMPLEADDRESS --node https://gravitychain.io:26657
```

## Sending tokens on Gravity Bridge to Ethereum

Sending tokens to Ethereum does not have a predictable timeline like sending tokens to Cosmos. In exchange for dramatically lower fees [relayers](relaying.md) will choose when to request a batch containing your tx and many other people's transactions to relay.

You can imagine Gravity Bridge batch fees as a metaphorical bus stop. Various people wishing to travel walk to the bus stop and put money into a jar. When enough money to pay for the bus driver (relayer) has been collected, the travelers board the bus and it departs for Ethereumville. You can supply any fee you like, all the way down to zero, in which case your tx will be picked up for free when other profitable transactions fill the metaphorical jar and make it worth the relayers time.

The bridge fee must be in the same token you are sending across the bridge.

Suggested fees:

* Someday: Zero fee
* Within a day: 50k Ethereum gas worth
* Within an hour: 200k Ethereum gas worth

The easiest way to run this is to paste the following in a text editor, modify, then copy/paste into a terminal. The first value is the amount you are sending, the second is the bridge fee being paid. The `--fees` argument is the transaction fee on the Gravity Bridge chain and can be denominated in any token the validators will accept.

```bash
gravity tx gravity send-to-eth gravityORIGIN_ADDRESS 0xDESTINATIONONETH <AMOUNTdenom to send e.g. 1000000ugraviton> 500ugraviton --node https://gravitychain.io:26657 --fees 0ugraviton --chain-id gravity-bridge-3
```

Once your tx is sent you can wait for a relayer to relay it. If you become impatient, it is possible to cancel the transaction and receive your funds back in your account on the Gravity Bridge chain side of the bridge.

### Canceling a send to ethereum

If your `send-to-eth` transaction has a fee that is too small, you may need to cancel the transaction and increase it. Alternatively, you may wish to continue using the tokens on Gravity Bridge. In either case you can cancel the send, and have the amount refunded to your account immediately.

First run the following:

```bash
gravity query gravity pending-send-to-eth gravity1YOURADDRESS
```

You will see a list of transfers, but you can only cancel `unbatched_transfers`. Transfers in batches will eventually time out (after 8 hours currently) if not relayed to Ethereum, at which point they will become available in the unbatched transfers list to cancel.

Once you have the transfer id, run the command:

```bash
gravity tx gravity cancel-send-to-eth TXID
```

Note that this transaction must be from the same private key as the one that sent the original `send-to-eth` message. Only the sender can cancel their own transaction.

As the final step, check your balance and confirm the tokens have been refunded. If you do not see the tokens check Ethereum and reconfirm that the transfer you have tried to cancel is in `unbatched_transfers`.

## Preparing Cosmos originated tokens to send to Ethereum

In order to send Cosmos-originated tokens to Ethereum there are three prerequisites.

1) An IBC channel must be established between your chain and Gravity Bridge
2) The token must have metadata set
3) An ERC20 representation must be deployed

In this context 'Cosmos originated' essentially means IBC tokens. Graviton is of course included with its metadata already set, as it is the native token.

Once all three of these conditions are met, they are permanent, and the token can be moved back and forth freely.

For more information on setting up an IBC channel between your chain and Gravity Bridge, see: [https://hermes.informal.systems/](https://hermes.informal.systems/)

### Setting metadata

First check if the token has metadata already set run:

```bash
gravity query bank denom-metadata --node https://gravitychain.io:26657
```

If metadata is not been set you will need to submit an ERC20MetadataProposal. Please see [custom governance proposals](custom-gov.md) for reference. Once this proposal has passed, you can move onto deploying the ERC20 contract.

### Deploying the ERC20 contract

This step involves deploying an ERC20 contract on Ethereum, the validators of the Gravity Bridge will then check this ERC20 and if it is valid, it will be adopted as the representation of this token-type.

You will need to provide an Ethereum key with enough Ethereum to cover the 700k gas required to deploy the contract. We suggest doing this early morning PST as this is when gas prices are lowest.

```bash
gbt client deploy-erc20-representation \
--cosmos-denom "denom of the token goes here, use the ibc/ or just graviton"
--ethereum-key "0xETHPRIVATEKEY" \
--gravity-contract-address "0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906" \
--ethereum-rpc https://eth.althea.net
--cosmos-grpc https://gravitychain.io:9090
```

Once your contract is deployed `gbt` will monitor the adoption process and let you know if the validators accept it. After that you can send the tokens to Ethereum following the instructions above.

## Sending tokens on Gravity Bridge to Osmosis via IBC

Gravity Bridge has an open connection to the Osmosis chain. You can find a full list of channels on Gravity Bridge [here](https://www.mintscan.io/gravity-bridge/relayers).

You can also use this [Keplr based guide](https://catdotfish.medium.com/getting-started-with-ibc-transfers-276e9ce91e17).

```bash
gravity tx ibc-transfer transfer transfer channel-10 osmosis1ADDRESS 1000000ugraviton --from yourkeyname --chain-id gravity-bridge-3
```
