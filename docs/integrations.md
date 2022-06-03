# Integrations

Gravity Bridge was designed to be a critical piece of infrastructure for the greater Cosmos ecosystem, thus the true power of Gravity comes from integrations with other Cosmos blockchains. This document notes important integration information meant for command line users, dApp developers, validators, and other highly technical users.

## **Evmos**

### IBC Channel

| Gravity Channel Id                                                       | Evmos Channel Id                                              |
| ------------------------------------------------------------------------ | ------------------------------------------------------------- |
| [channel-65](https://www.mintscan.io/gravity-bridge/relayers/channel-65) | [channel-8](https://www.mintscan.io/evmos/relayers/channel-8) |

### Tokens

| Name                                                                                 | Gravity Denom                                     | Evmos Denom                                                          | Evmos ERC20 Address                        | Token Decimals |
| ------------------------------------------------------------------------------------ | ------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------ | -------------- |
| GRAV                                                                                 | ugraviton                                         | ibc/7F0C2CB6E79CC36D29DA7592899F98E3BEFD2CF77A94340C317032A78812393D | 0x80b5a32E4F032B2a058b4F29EC95EEfEEB87aDcd | 6              |
| [DAI](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f)         | gravity0x6B175474E89094C44Da98b954EedeAC495271d0F | ibc/F96A7F81E8F82E4EE81F94D507CD257319EFB70FE46E23B4953F63B62E855603 | 0xd567B3d7B8FE3C79a1AD8dA978812cfC4Fa05e75 | 18             |
| [USDC](https://etherscan.io/token/0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48)        | gravity0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 | ibc/693989F95CF3279ADC113A6EF21B02A51EC054C95A9083F2E290126668149433 | 0x5FD55A1B9FC24967C4dB09C513C3BA0DFa7FF687 | 6              |
| [USDT](https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7)        | gravity0xdAC17F958D2ee523a2206206994597C13D831ec7 | ibc/DF63978F803A2E27CA5CC9B7631654CCF0BBC788B3B7F0A10200508E37C70992 | 0xecEEEfCEE421D8062EF8d6b4D814efe4dc898265 | 6              |
| [Wrapped BTC](https://etherscan.io/token/0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599) | gravity0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599 | ibc/350B6DC0FF48E3BDB856F40A8259909E484259ED452B3F4F39A0FEF874F30F61 | 0x1D54EcB8583Ca25895c512A8308389fFD581F9c9 | 8              |
| [Wrapped ETH](https://etherscan.io/token/0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2) | gravity0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 | ibc/6B3FCE336C3465D3B72F7EFB4EB92FC521BC480FE9653F627A0BD0237DF213F3 | 0xc03345448969Dd8C00e9E4A85d2d9722d093aF8E | 18             |

#### Adding Evmos ERC20 Addresses to MetaMask

Please see [the Evmos documentation to add Evmos to MetaMask](https://docs.evmos.org/users/wallets/metamask.html)

With Evmos Mainnet selected as the network and your Evmos account selected in Metamask, scroll down to the bottom of the Assets tab and you should see "Don't see your token?" and a "Import tokens" link to click.

On the Import Tokens page, paste one of the above Evmos ERC20 Addresses into the "Token Contract Address" field, and the rest of the token information should auto-populate. Next click "Add Custom Token" at the bottom and finally "Import tokens" on the next page.

### **I transferred Gravity Bridge tokens to Evmos via IBC, but they aren't appearing in MetaMask, what do I do?**

Evmos requires IBC tokens to be converted into ERC20 tokens before they can be used, run the following Evmos command to move your tokens into the EVM.

`evmosd --node https://tendermint.bd.evmos.org:26657 tx erc20 convert-coin <amount><Evmos Denom> <EVM Receiver> --from <Cosmos Sender> --chain-id evmos_9001-2 --gas 10000000`

> \<amount\> will be in terms of the base denom, so when converting GRAV to ERC20 coins, an amount of 1 will be 1 ugraviton, which is 0.000001 GRAV. For WETH this would be 1 wei, and for WBTC this would be 1 satoshi. Note that no space should be between \<amount\> and \<Evmos Denom\>
> \<Evmos Denom\> should be one of the above ibc/\<HASH\> addresses
> \<EVM Receiver\> should be a "0x" prefixed EIP-55 address, such as [this one](evm.evmos.org/address/0xA61808Fe40fEb8B3433778BBC2ecECCAA47c8c47). Use `evmosd debug addr <Cosmos Address>` to find your EIP-55 address.
> \<Cosmos Sender\> should be a "evmos1" prefixed Bech32 Acc address (or your keyname registered with evmosd), such as [this one](https://www.mintscan.io/evmos/account/evmos15cvq3ljql6utxseh0zau9m8ve2j8erz89m5wkz). Use `evmosd debug addr <EVM Address>` to find your Bech32 Acc address.

### **I have Gravity Bridge tokens on Evmos as ERC20s, but I want to move them back to Gravity Bridge, to another Cosmos chain, or to Ethereum, what should I do?**

To convert your Evmos ERC20s back to IBC tokens, you can run the following command:

`evmosd --node https://tendermint.bd.evmos.org:26657 tx erc20 convert-erc20 <Evmos ERC20 Address> <amount> <Cosmos Receiver> --from <Cosmos Sender> --chain-id evmos_9001-2 --gas 10000000`

> \<amount\> will be in terms of the base denom, so when converting ERC20-GRAV to IBC-GRAV, an amount of 1 will be 1 ugraviton, which is 0.000001 GRAV. For WETH this would be 1 wei, and for WBTC this would be 1 satoshi.
> \<Evmos ERC20 Address\> should be one of the above addresses
> \<Cosmos Receiver\> should be a "evmos1" prefixed Bech32 Acc address such as [this one](https://www.mintscan.io/evmos/account/evmos15cvq3ljql6utxseh0zau9m8ve2j8erz89m5wkz). Use `evmosd debug addr <EVM Address>` to find your Bech32 Acc address.
> \<Cosmos Sender\> should be a "evmos1" prefixed Bech32 Acc address (or your keyname registered with evmosd), such as [this one](https://www.mintscan.io/evmos/account/evmos15cvq3ljql6utxseh0zau9m8ve2j8erz89m5wkz). Use `evmosd debug addr <EVM Address>` to find your Bech32 Acc address.
