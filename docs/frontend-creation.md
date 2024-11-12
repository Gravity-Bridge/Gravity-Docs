# Creating a Gravity Bridge frontend

This document covers how to interact with Gravity Bridge as a frontend integrator.

There are two major supported usecases, a 'local' frontend will communicate directly with Gravity Bridge RPC and provide the most visibility into the workings of the bridge.

A 'remote' frontend only interacts with a destination blockchain, this chain will have an IBC channel to Gravity Bridge and you as an application developer will be able to send tokens to and from Ethereum without having to interact with Gravity Bridge RPC.

Please also see the [Gravity Info API](https://github.com/Gravity-Bridge/gravity-info-api) which provides a API for getting Gravity Bridge status information.

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

For [events](https://docs.cosmos.network/master/core/events.html) a websocket protocol endpoint is available at `wss://gravitychain.io:26657`. Contrary to the events documentation typed events are deployed and in use on Gravity Bridge. When it comes to interpreting these events you will need to have the proto file definitions handy.

The server at `gravitychain.io` also provides ABCI and Legacy Amino endpoints see the [resources](resources.md) doc for more details.

## Sending tokens from Ethereum to Gravity Bridge

In order to interact with the Gravity Bridge Ethereum contract you will need to initialize the users [Ethereum wallet](https://docs.metamask.io/guide/ethereum-provider.html#table-of-contents) then using the information provided by the user [encode the contract call](https://docs.metamask.io/guide/initializing-dapps.html#the-contract-network) before finally [sending a transaction](https://docs.metamask.io/guide/sending-transactions.html#example) with the encoded contract call set as the transaction bytes.

The Gravity Bridge contract ABI is available as `Gravity.json` attached to every [release](https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.8.1/Gravity.json)

[0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906](https://etherscan.io/address/0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906) is the Gravity Bridge contract address.

The first transaction you will have to send is an ERC20 `approve()` tx. This will allow the Gravity Bridge contract to remove the ERC20 token from the users wallet and lock it in the bridge. To do this you will need to load the ERC20 contract standard ABI, encode the approve, and send it on behalf of the user with the Gravity Bridge contract address as a parameter. If you do not do this the Gravity Bridge contract will not be able to lock up the users ERC20 tokens.

Once `approve()` has passed the next (and last step) is to call [SendToCosmos](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/solidity/contracts/Gravity.sol#L555) call takes

```solidity
 function sendToCosmos(
  address _tokenContract,
  string calldata _destination,
  uint256 _amount
```

This will lock up `amount` of hte users ERC20 tokens as identified by `tokenContract` value. The `destination` is a bech32 encoded Gravity address in the form of `gravity1<HASH>`.

Once the transaction has been submitted and succeeded the tokens should be available on Gravity Bridge within `96` Ethereum blocks. This is about 20 minutes. This delay is for finality and will be removed once Ethereum moves to proof of stake.

In the Gravity Bridge events section we discuss some events that provide more insight into how this deposit is being processed, the only event fired by a deposit on the Ethereum side is the [SendToCosmosEvent](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/solidity/contracts/Gravity.sol#L96) which tells you that your Ethereum transaction succeeded and starts the fifty block delay period.

The section [Monitoring a send from Ethereum to Gravity Bridge](#monitoring-a-send-from-ethereum-to-gravity-bridge) contains more info about how to monitor the process from here. But for the most part this won't be required, the tokens will be available on Gravity Bridge in less than 20 minutes with no further action from the user.

### Metamask Example

See the [Metamask docs](https://docs.metamask.io/guide/sending-transactions.html#example) and [Web3 docs](https://web3js.readthedocs.io/en/v1.2.11/getting-started.html) for more context. Note that you could easily use Web3.js to send the transaction if you don't need Metamask to access keys. This code will create your transaction and prompt the user to sign and send.

Once this tx is broadcasted on Ethereum tokens will be delivered directly to the target address on Gravity Bridge

```javascript
// Gravity Bridge contract address
let gravityBridge = "0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906"
// USDC ERC20 contract address
let erc20 = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"
let destination = "gravity18xvpj53vaupyfejpws5sktv5lnas5xj2kwracl"
// amount of USDC to send, since USDC has 6 decimals divide any value here by 1*10^decimals for the
// whole number of USDC. You should use the ERC20 decimals() call for this since tokens have different values
let amount = 50000

// approve must only be called once on the ERC20, in this example
// we limit the amount sent to the exact user send. In practice you may
// want to set a very large amount to save on gas re-sending a new approval
// each time.
let approve = web3.eth.abi.encodeFunctionCall({
    name: 'approve',
    type: 'function',
    inputs: [
    {
        type: 'address',
        name: 'spender'
    },
    {
        type: 'uint256',
        name: 'amount'
    }
    ]
}, [erc20, amount]);
// the actual send to cosmos function call that will lock
// the tokens for transfer
let sendToCosmos = web3.eth.abi.encodeFunctionCall({
    name: 'sendToCosmos',
    type: 'function',
    inputs: [
    {
        type: 'address',
        name: '_tokenContract'
    },
    {
        type: 'string',
        name: '_destination'
    },
    {
        type: 'uint256',
        name: '_amount'
    }
    ]
}, [erc20, destination, amount]);
const approveTransactionParameters = {
  to: erc20,
  from: ethereum.selectedAddress,
  value: '0x00',
  data: approve,
};
const txHash = await ethereum.request({
  method: 'eth_sendTransaction',
  params: [approveTransactionParameters],
});
const sendTransactionParameters = {
  to: gravityBridge,
  from: ethereum.selectedAddress,
  value: '0x00',
  data: sendToCosmos,
};
const txHash = await ethereum.request({
  method: 'eth_sendTransaction',
  params: [sendTransactionParameters],
});
```

## Sending tokens from Gravity Bridge to Ethereum

Gravity Bridge works on a batching model for transactions to Ethereum. 

```
User calls sendToEth() for token X on Gravity
User calls requestBatch() on Gravity, all waiting transactions for token X are placed in a single batch
User calls submitBatch() on Gravity.sol on Ethereum, unlocking all the tokens at once and paying the sum of fees to msg.sender
```

This model provides the possibility of dramatically lower fees. The base cost of verifying the validator signatures is about 400,000 Ethereum gas. Each additional transfer adds only about 10,000 gas. Exact totals depend on the ERC20 contract in question. But the advantage is clear, even if only two transactions share a batch the effective gas cost paid by both is halved.

Obviously the problem with sharing a batch is the time the user has to wait for someone else to show up to share the batch with. This unpredictable delay between calling sendToEth() on Gravity and the tokens becoming avaialble on Ethereum can be extremely frustrating to users if not communicated carefully.

### Basic Gravity Bridge to Ethereum flow

If you only have time to build the simpliest possible interface, we recomend hiding batching from users entierly. Prompt the user 3 times, twice with a Cosmos wallet (Keplr or Leap) to sign sendToEth() and requestBatch(), then once with Metamask. The user will need both whatever token they wish to bridge out on Gravity as well as ETH to pay the fees on Ethereum. All fees on Gravity are paid in the token the user is bridging out.

[MsgSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/msgs.proto#L120) 

```
message MsgSendToEth {
  string                   sender   // The gravity1 encoded address of the sending account
  string                   eth_dest // The 0x encoded Ethereum address of the destination account (EIP-55 capitalization required)
  cosmos.base.v1beta1.Coin amount   // The amount and token type being sent
  cosmos.base.v1beta1.Coin bridge_fee // This amount is locked up to pay whomever submits the submitBatch() call on Ethereum
  cosmos.base.v1beta1.Coin chain_fee // This amount is distributed to Gravity Bridge stakers and the auction module
}
```

The amount, bridge_fee and chain_fee must all be of the same token type. 
The chain_fee must be greater than or equal to the value of the AMOUNT field times the [MinChainFeeBasisPoints](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/x/gravity/types/genesis.go#L88) parameter (convert to a decimal by dividing by 10,000 first).
The bridge_fee value can be any amount of the token being sent, including zero.

Note that the above two fees are inside the transaction itself. The MsgSendToEth transaction itself has the usual fee field like any other Gravity Bridge transaction. This fee field can be any token accepted by the validator. You can use 0 of the token you are sending for this fee field since Gravity Bridge valdiators currently do have a mininum fee for transactions.

In order to create the MsgSendToEth you should query the [MinChainFeeBasisPoints](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/x/gravity/types/genesis.go#L88) parameter. DO NOT HARDCODE THIS VALUE, GOVERNANCE MAY CHANGE THE MIN FEE REGULARLY. At which point users of your frontend will either overpay if the min fee was lowered or be unable to bridge out if it was increased.

chain_fee = amount * MinChainFeeBasisPoints / 10000
total_token_cost = amount + chain_fee + bridge_fee + gravity transaction fee

For the sake of providing the simplest possible functional example you only need to provide the chain_fee. Set the other fees to zero.

Once you have called sendToEth() the users transaction will be visable via [this api call](https://github.com/Gravity-Bridge/gravity-info-api?tab=readme-ov-file#gravity_bridge_info) or [this query](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L302)

Next you must call requestBatch()

[MsgRequestBatch](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/msgs.proto#L136)

```
message MsgRequestBatch {
  string sender // The gravity1 encoded address of the sending account
  string denom  // the denom of the token being bridged either gravity0xER20ADDRESS or ibc/IBCHASH
}
```

This request will bundle all transactions for the requested token type, including the users transaction that was just sent with sendToEth().

It is possible for MsgRequestBatch to fail, this is an anti-spam measure that only occurs when there is a higher fee batch waiting, but not yet relayed. There are three ways to handle this edgecase. 

1. By [querying pending batches](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L163) before having the user call sendToEth you can check if a batch of the same token type exists, and if it does simply specify a slightly higher fee for the bridge_fee field.
2. Also query pending batches, but look at the eth timeout block for the batch that is blocking the users request and inform the user to wait for that Ethereum block. Obviously this is less ideal than (1).

Finally now that the users transaction can be relayed to Ethereum.

This ETH transaction will call submitBatch() on [Gravity.sol](https://etherscan.io/address/0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906#code#F1#L344). [This api call](https://github.com/Gravity-Bridge/gravity-info-api?tab=readme-ov-file#batch_txbatch_nonce) will build the data field of the Ethereum transaction. 

```javascript
// Gravity Bridge contract address
let gravityBridge = "0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906"
let apiCallBytes = ...

// sign and send the transaction
const receipt = await web3.eth.sendTransaction({
  from: userAddress,
  to: gravityBridge,
  data: apiCallBytes
  value: 0,
});
```

If you wish to build the data field yourself, you will need to search the event history on Gravity.sol for the most recent [ValsetUpdatedEvent](https://etherscan.io/address/0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906#code#F1#L112). Then query on the Gravity Bridge side for the [batch confirmation signatures](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L180). Then encode both sets of data into the [submitBatch()](https://etherscan.io/address/0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906#code#F1#L344) call.

Once the user signs this ETH transaction and pays the required Ethereum gas their tokens will be immediately avaialble on ETH. Note that it is possible that the user will actually recieve more tokens than they sent, if the batch contains other transactions with non-zero fees. If this is the case you should recomend the user use an MEV protected rpc endpoint to relay the transaction. Otherwise the batch profits will be sniped by an MEV bot. Being MEV sniped will not interfere with the users sendToEth() transaction, just with recieving the bridge_fee from other users. So it's safe to ignore this. 

### Expanded Gravity to Ethereum flow

In order to fully take advantage of Gravity Bridge's batching infrastructure you can present the user the option to pay a much smaller bridge_fee than they would pay in Ethereum gas for their transaction to execute 'eventually'. Users may frequently decide they are willing to wait, and only a few minutes later decide they are not so willing to wait. In order to respond to this we suggest the following flow.

When the user goes to bridge out present a 'wait and save' option as well as an 'instant' option. If the user selects instant run through the flow described in the previous section. If the user selects 'wait and save' create a `MsgSendToEth` transaction and set the bridge_fee to the value of 50K Ethereum gas but in the token the user is bridging.

This step above requires a price lookup for the bridged token and an Ethereum gas lookup. If you can not lookup a price for the token being bridged, fall back to the instant flow or allow the user to specify the bridge_fee tokens directly. It's important to handle this failure condition well as price lookups can be especially failure prone.

Once the user has submitted their sendToEth() transaction the tokens will no longer be visible in their account. At that point you *MUST* display the users pending tokens by querying [this api call](https://github.com/Gravity-Bridge/gravity-info-api?tab=readme-ov-file#gravity_bridge_info). If you do not display the users tokens waiting to be bridged they will become confused and concerned.

Next to the display for the users pending tokens you can provide an instant relay button. This button should skip to the requestBatch() part of the previous flow. Allowing the user to transition from waiting for their batched transaction to be relayed, to performing the instant relay flow. 

Optionally you can also allow the user to send a [MsgCancelSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/msgs.proto#L279)

```
message MsgCancelSendToEth {
  uint64 transaction_id // the internal transaction id for the sendToEth() transaction, this value is provided by the pendingSendtoEth query https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L302
  string sender // the gravity1 address of the sender
}
```

This will refund the user their tokens instantly, if and only if their transaction has not yet been batched. Once the transaction has been batched it may or may not be relayed to Ethereum and to avoid double spends the user will have to wait until that batch times out and that timeout has been confirmed to cancel it.

Please see the sections, for more info.

[Monitoring a send from Gravity Bridge to Ethereum](#monitoring-a-send-from-gravity-bridge-to-ethereum)
[Canceling a send from Gravity Bridge to Ethereum](#canceling-a-send-from-gravity-bridge-to-ethereum)

## Sending tokens from Ethereum through Gravity Bridge to another chain

This feature has been available since the Mercury upgrade and is extremely straightforward to use.
Following the steps outlined in [Sending tokens from Ethereum to Gravity Bridge](#sending-tokens-from-ethereum-to-gravity-bridge) simply replace the `gravity1<hex>` bech32 encoded address with the chain prefix of your choice for example a deposit to `osmosis1` will go to Osmosis and `cosmos1` will go to the Cosmos Hub.

This does require some setup though. Before you can use this feature you must make sure the address prefix you are interested in using has been mapped to an IBC channel using the command `gravity tx bech32ibc update-hrp-ibc-record` to create a governance proposal

Once this proposal has passed simply call `Gravity.sol` `sendToCosmos` endpoint with a native address (for example one starting with `cosmos1`) and the transaction will be instantly forwarded.

### Metamask Example Remote Send

See the [Metamask docs](https://docs.metamask.io/guide/sending-transactions.html#example) and [Web3 docs](https://web3js.readthedocs.io/en/v1.2.11/getting-started.html) for more context. Note that you could easily use Web3.js to send the transaction if you don't need Metamask to access keys. This code will create your transaction and prompt the user to sign and send.

Once this tx is broadcasted on Ethereum tokens will be delivered directly to the target chain as routed via the prefix.

```javascript
// Gravity Bridge contract address
let gravityBridge = "0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906"
// USDC ERC20 contract address
let erc20 = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"
// destination using destination chain prefix. If the prefix has not been configured
// for forwarding the address will be re-prefixed to gravity and funds will be left
// there for the user to pick up
let destination = "osmo18xvpj53vaupyfejpws5sktv5lnas5xj2kwracl"
// amount of USDC to send, since USDC has 6 decimals divide any value here by 1*10^decimals for the
// whole number of USDC. You should use the ERC20 decimals() call for this since tokens have different values
let amount = 50000

// approve must only be called once on the ERC20, in this example
// we limit the amount sent to the exact user send. In practice you may
// want to set a very large amount to save on gas re-sending a new approval
// each time.
let approve = web3.eth.abi.encodeFunctionCall({
    name: 'approve',
    type: 'function',
    inputs: [
    {
        type: 'address',
        name: 'spender'
    },
    {
        type: 'uint256',
        name: 'amount'
    }
    ]
}, [erc20, amount]);
// the actual end to comsos function call that will lock
// the tokens for transfer
let sendToCosmos = web3.eth.abi.encodeFunctionCall({
    name: 'sendToCosmos',
    type: 'function',
    inputs: [
    {
        type: 'address',
        name: '_tokenContract'
    },
    {
        type: 'string',
        name: '_destination'
    },
    {
        type: 'uint256',
        name: '_amount'
    }
    ]
}, [erc20, destination, amount]);
const approveTransactionParameters = {
  to: erc20,
  from: ethereum.selectedAddress,
  value: '0x00',
  data: approve,
};
const txHash = await ethereum.request({
  method: 'eth_sendTransaction',
  params: [approveTransactionParameters],
});
const sendTransactionParameters = {
  to: gravityBridge,
  from: ethereum.selectedAddress,
  value: '0x00',
  data: sendToCosmos,
};
const txHash = await ethereum.request({
  method: 'eth_sendTransaction',
  params: [sendTransactionParameters],
});
```

## Sending tokens from another chain through Gravity Bridge and to Ethereum

This feature will require interchain accounts to be integrated into Gravity Bridge and the destination chain you will be integrating against. Gravity Bridge does not currently have interchain accounts support.

## Monitoring a send from Gravity Bridge To Ethereum

The lifecycle of a [MsgSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/msgs.proto#L101) transaction is as follows.

- User sends MsgSendToEth, removing the tokens from their balance
- The message hander checks if the user has submitted a valid value for the CHAIN_FEE field. The fee must be
  greater than or equal to the value of the AMOUNT field times the [MinChainFeeBasisPoints](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/x/gravity/types/genesis.go#L88) parameter. If a MsgSendToEth fails this check it will be rejected and user funds will not be withdrawn. The BRIDGE_FEE value can be any
  amount of the token being sent, including zero.
- the transaction enters the SendToEth transaction pool for that token type
- a relayer looks at the total fees for the transaction pool for that token type
- a relayer requests a batch be created
- validators sign the batch
- the batch can then be relayed, or will time out in about 2 hours, returning the MsgSendToEth to the first step

Any time the MsgSendToEth transaction is in the transaction pool, and not in a signed batch, it is possible to [cancel the send](#canceling-a-send-from-gravity-bridge-to-ethereum) and have the user get an instant refund.

Once a MsgSendToEth is in a batch it is possible that it has been submitted to Ethereum, to prevent double spending it is not possible to cancel the send until the batch has timed out. This timeout value is a number of blocks and is set in the Gravity params. Due to the complexity of knowing exactly what block it is on Ethereum at any given time the batch will not complete timing out until any Gravity Bridge event is processed by the oracle with a later block height. For example a deposit of an unrelated token will complete a batch timeout by proving to Gravity Bridge that the Ethereum block height has exceeded the batch timeout height.

You can use the [Gravity Info API](https://github.com/Gravity-Bridge/gravity-info-api) to easily grab batch and event info.

In order to determine the status of a user transaction you should check.

- if the users transaction is in a batch
- the total fee value for the transaction pool of that token type

This will let you display if the user can cancel their transaction, and predict at least to some degree when a batch might be created and relayed.

[QueryPendingSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L260) will return all MsgSendToEth transactions sent by this user and indicate if they are currently in a batch.

[QueryBatchFeeRequest](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L139) will return a list of fee totals for all token types currently being bridged.

The Info API returns the data from both of these interal queries at the endpoint `https://info.gravitychain.io:9000/gravity_bridge_info` and you can use `https://info.gravitychain.io:9000/erc20_metadata` to get up to date exchange rates for any bridged token.

Batches cost about 600k GAS to execute. When you see a pending batch or a batch with that gas value in fees relaying is immenent.

Finally we have to detect when a batch is actually relayed.

Depending on which is easier for you there are two methods to do this.

- Monitor `Gravity.sol` for a [TransactionBatchExecutedEvent](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/solidity/contracts/Gravity.sol#L91)
- Use [QueryAttestationsRequest](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L217) to monitor for a `BatchSendToEthClaim`

Or `https://info.gravitychain.io:9000/gravity_bridge_info` and `https://info.gravitychain.io:9000/eth_bridge_info` respectively.

Either will allow you to monitor batch execution and confirm to the user that their funds are available on Ethereum, either by finding the batch in the recently executed events list on the ETH side or seeing it go away from the Gravity Bridge side.

## Monitoring a send from Ethereum to Gravity Bridge

Once a `MsgSendToCosmos` transaction has been included in an Ethereum block the process for the rest of the deposit is hands off for the user.

In order to display progress to the user you should.

- Query the txid and show the user that the transaction is waiting on Ethereum
- Once the transaction is included in a block display a 96 block countdown until funds are available on Gravity Bridge

The `https://info.gravitychain.io:9000/eth_bridge_info` endpoint displays seconds until confirmation for deposit events, you can simply query this info.

## Canceling a send from Gravity Bridge To Ethereum

In order to cancel a MsgSendToEth you must first check if the transaction is in a batch using.

[QueryPendingSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/query.proto#L260).

Or `https://info.gravitychain.io:9000/gravity_bridge_info` which will display what batches are avaialble, their transactions and the id's of those transactions

If the MsgSendToEth is in a batch, you must wait for that batch to time out before canceling is possible. The batch may also be executed, at which point the tokens are on Ethereum and canceling the operation is not possible.

Once you have determined that the MsgSendToEth is not in a batch you can retrieve it's `transaction_id` and send a [MsgCancelSendToEth](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/module/proto/gravity/v1/msgs.proto#L258) which will instantly cancel the transaction and refund the user.
