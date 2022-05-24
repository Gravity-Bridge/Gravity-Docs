# Creating ERC20 representations

One of the main features of Gravity Bridge is to provide an ERC20 representation of a given CosmosSDK token on Ethereum. Once bridged to Ethereum you can use the Cosmos token however you would use any other ERC20.

This opens up exiting use cases like listing Cosmos originated tokens in Uniswap or using them with other DeFi applications.

## Requirements

1. An IBC channel must be established between your chain and Gravity Bridge
1. The token must have metadata set
1. An ERC20 representation must be deployed

In this context 'Cosmos originated' essentially means IBC tokens. Graviton is of course included with its metadata already set, as it is the native token.

Once all three of these conditions are met, they are permanent, and the token can be moved back and forth freely.

## Steps to create a representation

1. Basic prep
1. Create an IBC channel
1. Submit an IBCMetadata governance proposal
1. Deploy the ERC20 contract on Ethereum
1. Send tokens to Ethereum

## Basic prep

1. Download [the latest release](https://github.com/Gravity-Bridge/Gravity-Bridge/releases) of Gravity and gbt (gravity bridge tools)
1. Have the minimum required GRAV to create a [governance proposal](https://www.mintscan.io/gravity-bridge/proposals) (10 at the time of writing)
1. Have enough Ethereum to cover 700k in gas costs (somewhere between $50 and $400 depending on time of day at the time of writing)

You will not need your own full node for either Gravity Bridge or Ethereum. Working endpoints are provided in this document. Infura or any other endpoint service will also work.

## Create an IBC channel

Creating an IBC channel is extensively documented outside of Gravity Bridge, no special channel parameters or settings are required.

For more information on setting up an IBC channel between your chain and Gravity Bridge, see: [https://hermes.informal.systems/](https://hermes.informal.systems/)

Once you have created the channel bridge any amount of the token you want to create a representation for, in order to see it's `ibc/hash` token name. You will need this value for the governance proposal next.

## Submit an IBCMetadata governance proposal

In order for the ERC20 token on Ethereum to have the correct symbol, name, decimals, and description metadata must be set for the token to be represented.

The `IBCMetadataProposal` allows for the governance to set the denom metadata for any IBC token in a decentralized manner.

Here is an example proposal.json

```json
{
 "title": "My proposal title",
 "description": "My proposal text, use \n to generate a new line",
 "metadata": {
  "base": "ibc/hashvalue",
  "denom_units": [
   {
    "aliases": [
     "uatom"
    ],
    "denom": "ibc/hashvalue",
    "exponent": 0
   },
   {
    "aliases": [
     "atom"
    ],
    "denom": "atom",
    "exponent": 6
   }
  ],
  "description": "The ATOM token",
  "display": "atom",
  "name": "Atom",
  "symbol": "ATOM"
 },
 "ibc_denom": "ibc/hashvalue"
}
```

Note a few key elements here. The base denom and the `ibc_denom` field must line up and be the IBC token name on the Gravity Bridge chain.

Once you have formed your `proposal.json` simply submit it

```bash
gravity tx gravity gov-ibc-metadata proposal.json 10000000ugraviton --from <key_name> --chain-id gravity-bridge-3
```

Once the proposal has passed confirm your metadata has been set

```bash
gravity query bank denom-metadata --node https://gravitychain.io:26657
```

## Deploy the ERC20 contract

Once the IBCMetadata proposal has passed the remaining steps are permissionless. Anyone can deploy the representative ERC20 contract at any time, Gravity Bridge will adopt the first contract to be deployed.

You will need to provide an Ethereum key with enough Ethereum to cover the 700k gas required to deploy the contract. We suggest doing this early morning PST as this is when gas prices are lowest.

```bash
gbt client deploy-erc20-representation \
--cosmos-denom "denom of the token goes here, use the ibc/hash or just graviton" \
--ethereum-key "0xETHPRIVATEKEY" \
--ethereum-rpc https://eth.althea.net \
--cosmos-grpc https://gravitychain.io:9090
```

Once your contract is deployed `gbt` will monitor the adoption process and let you know if the validators accept it. If they do the ERC20 address will be printed.

## Send tokens to Ethereum

Now that the ERC20 representation is deployed it's time to actually use the newly created ERC20.

The following command will put some tokens into the batch pool. It will take some time for a relayer to relay them, once the token is listed in Uniswap relayers will automatically relay profitable batches right away.

```bash
gravity tx gravity send-to-eth gravityORIGIN_ADDRESS 0xDESTINATIONONETH 1000000ibc/hash 500ibc/hash --node https://gravitychain.io:26657 --fees 0ugraviton --chain-id gravity-bridge-3
```
