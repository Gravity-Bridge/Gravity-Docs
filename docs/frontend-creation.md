# Creating a Gravity Bridge frontend

This document covers how to interact with Gravity Bridge as a frontend integrator.

There are two major supported usecases, a 'local' frontend will communicate directly with Gravity Bridge RPC and provide the most visibility into the workings of the bridge.

A 'remote' frontend only interacts with a destination blockchain, this chain will have an IBC channel to Gravity Bridge and you as an application developer will be able to send tokens to and from Ethereum without having to interact with Gravity Bridge RPC.

## Index

[Getting started](#getting-started-with-cosmossdk-development)

[Sending tokens from Ethereum to Gravity Bridge](#sending-tokens-from-ethereum-to-gravity-bridge)

[Sending tokens from Gravity Bridge to Ethereum](#sending-tokens-from-gravity-bridge-to-ethereum)

[Sending tokens from Ethereum through Gravity Bridge to another chain](#sending-tokens-from-ethereum-through-gravity-bridge-to-another-chain)

[Sending tokens from another chain through Gravity Bridge to Ethereum](#sending-tokens-from-another-chain-through-gravity-bridge-and-to-ethereum)

[Monitoring a send from Ethereum to Gravity Bridge](#monitoring-a-send-from-ethereum-to-gravity-bridge)

[Monitoring a send from Gravity Bridge to Ethereum](#monitoring-a-send-from-gravity-bridge-to-ethereum)

[Canceling a send from Gravity Bridge to Ethereum](#canceling-a-send-from-gravity-bridge-to-ethereum)

## Getting started with CosmosSDK development

This section tries to provide a very basic best-effort guide at getting started with CosmosSDK frontend development. It is by no means completely but is hopefully helpful.

[Keplr](https://www.keplr.app/) is by far the most common wallet for end users.

Keplr with Ledger support is the gold standard for UX + Security at the moment.

- [Keplr API](https://docs.keplr.app/api/#how-to-detect-keplr)

- [Using Keplr with CosmJs](https://docs.keplr.app/api/cosmjs.html)

CosmosSDK currently has [3 distinct RPC](https://docs.cosmos.network/v0.42/core/grpc_rest.html) endpoints that can be used to query data and
send transactions

- [Legacy Amino](https://docs.cosmos.network/v0.42/migrations/rest.html) which is deprecated and will hopefully be removed soon
- [ABCI](https://cosmos-network.gitbooks.io/cosmos-academy/content/cosmos-for-developers/tendermint/abci-protocol.html) ABCI is poorly documented but is heavily used by most clients
- [gRPC](https://docs.cosmos.network/v0.42/core/proto-docs.html) is the modern endpoint. If you're starting development now you should use gRPC, the question is exactly how.

We provide gRPCWeb support at `https://gravitychain.io:9091`, gRPC web [has a pretty steep setup curve](https://grpc.io/docs/platforms/web/) mainly revolving around the proto file definitions.

In our documentation we will provide some REST compatible ABCI calls to query basic information, but any more advanced querying of Gravity Bridge should take the time to setup gRPC web, it will pay off very quickly if you are trying to process and interpret many large and complex data structures at once.

For [events](https://docs.cosmos.network/master/core/events.html) a websocket protocol endpoint is available at `wss://gravitychain.io:26657]`. Contrary to the events documentation typed events are deployed and in use on Gravity Bridge. When it comes to interpreting these events you will need to have the proto file definitions handy.

## Sending tokens from Ethereum to Gravity Bridge

In order to interact with the Gravity Bridge Ethereum contract you will need to initialize the users [Ethereum wallet](https://docs.metamask.io/guide/ethereum-provider.html#table-of-contents) then using the information provided by the user [encode the contract call](https://docs.metamask.io/guide/initializing-dapps.html#the-contract-network) before finally [sending a transaction](https://docs.metamask.io/guide/sending-transactions.html#example) with the encoded contract call set as the transaction bytes.

The Gravity Bridge contract ABI is available as `Gravity.json` attached to every [release](https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.4.1/Gravity.json)

[0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906](https://etherscan.io/address/0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906) is the Gravity Bridge contract address.

The first transaction you will have to send is an ERC20 `approve()` tx. To do this you will need to load the ERC20 contract standard ABI, encode the approve, and send it on behalf of the user with the Gravity Bridge contract address as a parameter. If you do not do this the Gravity Bridge contract will not be able to lock up the users ERC20 tokens.

Once `approve()` has passed the next (and last step) is to call [SendToCosmos](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/solidity/contracts/Gravity.sol#L555) call takes

```solidity
 function sendToCosmos(
  address _tokenContract,
  string calldata _destination,
  uint256 _amount
```

This will lock up `amount` of hte users ERC20 tokens as identified by `tokenContract` value. The `destination` is a bech32 encoded Gravity address in the form of `gravity1<HASH>`.

Once the transaction has been submitted and succeeded the tokens should be available on Gravity Bridge within `50` Ethereum blocks. This is about 10 minutes. This delay is for finality and will be removed once Ethereum moves to proof of stake.

In the Gravity Bridge events section we discuss some events that provide more insight into how this deposit is being processed, the only event fired by a deposit on the Ethereum side is the [SendToCosmosEvent](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/solidity/contracts/Gravity.sol#L96) which tells you that your Ethereum transaction succeeded and starts the fifty block delay period.

The section [Monitoring a send from Ethereum to Gravity Bridge](#monitoring-a-send-from-ethereum-to-gravity-bridge) contains more info about how to monitor the process from here. But for the most part this won't be required, the tokens will be available on Gravity Bridge in less than 20 minutes with no further action from the user.

## Sending tokens from Gravity Bridge to Ethereum

Sending tokens to Ethereum does not have a predictable timeline like sending tokens to Cosmos. In exchange for dramatically lower fees [relayers](relaying.md) will choose when to request a batch containing your users tx and many other people's transactions to relay.

You can imagine Gravity Bridge batch fees as a metaphorical bus stop. Various people wishing to travel walk to the bus stop and put money into a jar. When enough money to pay for the bus driver (relayer) has been collected, the travelers board the bus and it departs for Ethereum. You can supply any fee you like, all the way down to zero, in which case the users tx will be picked up for free when other profitable transactions fill the metaphorical jar and make it worth the relayers time.

The bridge fee must be in the same token you are sending across the bridge.

Suggested fees:

- Someday: Zero fee
- Within a day: 50k Ethereum gas worth
- Within an hour: 200k Ethereum gas worth
- Instantly: 500k Ethereum gas worth

In order to convert these values to ETH you should query the `eth_gasPrice` endpoint, and multiply the resulting value by the suggested values above.

```math
fee_cost_in_wei = eth_gasPrice * gasAmount 
```

This will give you a gas cost in wei, convert this value to the equivalent cost in the bridged token using a price oracle.

Once the user has selected their fee amount you can actually form and submit the [MsgSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/msgs.proto#L101) transaction.

Keep in mind

- `bridge_fee` must be the same token as `amount`
- This transaction still has a 'fee' field for Gravity Bridge fees, this can be set to zero for now
- `eth_dest` should be a correctly capitalized eip-55 address

Once the user has sent this message the tokens will be removed from their account, but they will not appear on Ethereum until the transaction batch is relayed.

Please see the sections, for more info.

[Monitoring a send from Gravity Bridge to Ethereum](#monitoring-a-send-from-gravity-bridge-to-ethereum)
[Canceling a send from Gravity Bridge to Ethereum](#canceling-a-send-from-gravity-bridge-to-ethereum)

## Sending tokens from Ethereum through Gravity Bridge to another chain

This feature will become available after the Mercury upgrade and is extremely straightforward to use.

Following the steps outlined in [Sending tokens from Ethereum to Gravity Bridge](#sending-tokens-from-ethereum-to-gravity-bridge) simply replace the `gravity1<hex>` bech32 encoded address with the chain prefix of your choice for example a deposit to `osmosis1` will go to Osmosis and `cosmos1` will go to the Cosmos Hub.

This does require some setup though. Before you can use this feature you must make sure the address prefix you are interested in using has been mapped to an IBC channel with a [UpdateHrpIbcChannelProposal](https://github.com/osmosis-labs/bech32-ibc/blob/master/proto/bech32ibc/v1beta1/gov.proto#L15).

TODO:

1. Add step by step guide for UpdateHrpIbcChannelProposal in custom-gov.md
1. Provide a way to check if a prefix has already been registered

## Sending tokens from another chain through Gravity Bridge and to Ethereum

This feature will require interchain accounts to be integrated into Gravity Bridge and the destination chain you will be integrating against. Gravity Bridge does not currently have interchain accounts support.

## Monitoring a send from Gravity Bridge To Ethereum

The lifecycle of a [MsgSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/msgs.proto#L101) transaction is as follows.

- User sends MsgSendToEth, removing the tokens from their balance
- the transaction enters the SendToEth transaction pool for that token type
- a relayer looks at the total fees for the transaction pool for that token type
- a relayer requests a batch be created when the gas price is low enough or the fees become high enough
- validators sign the batch
- the batch can then be relayed, or will time out in about 8 hours, returning the MsgSendToEth to the first step

Any time the MsgSendToEth transaction is in the transaction pool, and not in a signed batch, it is possible to [cancel the send](#canceling-a-send-from-gravity-bridge-to-ethereum) and have the user get an instant refund.

Query endpoints related to batches and essentially all internal functions of Gravity Bridge are currently GRPC only. We will be adding ABCI query endpoints for commonly used functionality but encourage use of gRPC-web where feasible.

In order to determine the status of a user transaction you should check.

- if the users transaction is in a batch
- the total fee value for the transaction pool of that token type

This will let you display if the user can cancel their transaction, and predict at least to some degree when a batch might be created and relayed.

[QueryPendingSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L260) will return all MsgSendToEth transactions sent by this user and indicate if they are currently in a batch.

[QueryBatchFeeRequest](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L139) will return a list of fee totals for all token types currently being bridged.

Translating QueryBatchFeeRequest into usable data takes a little work. First you must locate the token that is being bridged in the response. Then take a look at the total fees for that batch, convert these total fees to equal value in WETH and finally estimate what the cost of executing the batch would be.

It's infeasible to create a perfect fee estimation, so go with this heuristic. A batch starts at 400,000 Ethereum gas, and goes up in cost by 16,000 Ethereum gas per transaction included.

So a batch with 50 transactions would estimate at 1,200,000 Ethereum gas.

This gas estimate can be multiplied by the current gas price and then compared to the total fee value in WETH to determine if a batch will be relayed soon.

The effectiveness of any time estimations will be greatly improved by a [historical gas price oracle](https://etherscan.io/gastracker#historicaldata) which will allow you to easily figure out when fee value and gas prices intersect.

Finally we have to detect when a batch is actually relayed.

Depending on which is easier for you there are two methods to do this.

- Monitor `Gravity.sol` for a [TransactionBatchExecutedEvent](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/solidity/contracts/Gravity.sol#L91)
- Use [QueryAttestationsRequest](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L217) to monitor for a `BatchSendToEthClaim`

Either will allow you to monitor batch execution and confirm to the user that their funds are available on Ethereum.

## Monitoring a send from Ethereum to Gravity Bridge

Once a `MsgSendToCosmos` transaction has been included in an Ethereum block the process for the rest of the deposit is hands off for the user.

In order to display progress to the user you should.

- Query the txid and show the user that the transaction is waiting on Ethereum
- Once the transaction is included in a block display a 50 block countdown until funds are available on Gravity Bridge

If you want to have even more insight into the process on Gravity Bridge you can use [QueryAttestationsRequest](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L217) to observe the inner workings fo the validators confirming the deposit. This will not start until after 50 blocks have elapsed and should take only 10-20 seconds start to finish.

Observing the attestation process is probably not worth it from a UI perspective, but you can if you like. The attestation query endpoint may be more useful to observe when a batch has been relayed.

## Canceling a send from Gravity Bridge To Ethereum

In order to cancel a MsgSendToEth you must first check if the transaction is in a batch using.

[QueryPendingSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L260).

If the MsgSendToEth is in a batch, you must wait for that batch to time out before canceling is possible. The batch may also be executed, at which point the tokens are on Ethereum and canceling the operation is not possible.

Once you have determined that the MsgSendToEth is not in a batch you can retrieve it's `transaction_id` and send a [MsgCancelSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/msgs.proto#L258) which will instantly cancel the transaction and refund the user.
