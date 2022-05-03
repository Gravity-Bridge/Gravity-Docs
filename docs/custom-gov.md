# Gravity Bridge specific governance proposals

If you are looking to create a proposal please post on the [Commonwealth](https://commonwealth.im/gravity-bridge) forum to gather sentiment
and advice.

Gravity Bridge has a series of custom governance proposal types. This documents what these proposal types are and how to submit them at a technical level.

## IBC metadata

The `IBCMetadataProposal` allows for the governance to set the denom metadata for any IBC token in a decentralized manner. This is essential for the deployment of ERC20 representations of these tokens on Ethereum, which will use the denom metadata for the name, symbol, and decimals fields of the ERC20 contract.

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
gravity tx gravity gov-ibc-metadata proposal.json 10000000ugraviton
```

## Airdrop

The `AirdropProposal` provides a decentralized way for governance to airdrop to up to 45,000 individual addresses on the Gravity Bridge chain.

Here is an example proposal.json

```json
{
 "title": "My proposal title",
 "denom": "ugraviton",
 "description": "My proposal text, use \n to generate a new line",
 "amounts": [100, 200],
 "recipients": ["gravity1FIRSTPERSON", "gravity1SECONDPERSON"]
}
```

Note a few key things about the `proposal.json`. The `Amounts` and `Recipients` fields are interpreted in parallel. So the first amount goes to the first address and so on. Second the `Denom` must be a base unit it's not metadata aware.

Any denom in the community pool can be airdropped.

Once you have formed your `proposal.json` simply submit it

```bash
gravity tx gravity gov-airdrop proposal.json 10000000ugraviton
```

## IBC auto forwarding

The `UpdateHrpIbcChannelProposal` allows for sends to go directly from Ethereum to a Cosmos destination chain. For example a send to a cosmos1 address string sent to `Gravity.sol` will arrive directly at that address on the cosmos hub without stopping by Gravity Bridge and forcing the user to send an IBC transaction

In order to map the address prefix to the ibc channel you must submit a proposal using the command

`gravity tx bech32ibc update-hrp-ibc-record`
