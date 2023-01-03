# Recovering tokens "bounced back" from Canto

When sending tokens from Ethereum to Canto via Gravity Bridge, it is possible to send to your Canto address before
generating a public key on Canto. When this happens, Canto will immediately send the tokens back to the address which
sent them. This becomes complicated when Gravity Bridge's IBC Auto Forwarding function is involved, as you likely have
never used the Gravity Bridge account the funds are "stuck" on.

## Understanding where the "missing" funds are

Gravity Bridge's IBC Auto Forwarding functionality allows users to send tokens from Ethereum directly to Cosmos chains
without normally needing to interact with the Gravity Bridge Blockchain.

First, you submit a Send To Cosmos transaction on Ethereum to the Gravity Bridge contract with a receiving address like `canto1abcdefg4567...`.
The [Canto Website](https://bridge.canto.io) will do this automatically for you.
Behind the scenes, the Gravity Bridge Validators relay that event to the Gravity Bridge Blockchain, and the chain will generate an IBC
transfer to Canto chain.

If anything unexpected happens during the IBC transfer, the funds will wind up on a Gravity
Bridge account which looks extremely similar to the Canto receiving address like `gravity1abcdefg4567...`, the only
differences being the prefix (`gravity` vs `canto`) and the last 6 characters (which are the checksum bits of the address).

If you bridged tokens to Canto without first generating a Canto public key on the [Canto Website](https://bridge.canto.io),
Canto sent them back to the `gravity1abcdefg4567...` address you probably have never used. This account is an Ethereum
account in disguise, and currently there are no good user interfaces to control this account with.

## Recovery Process

1. Generate a public key on the [Canto Website](https://bridge.canto.io), otherwise you'll wind up with the tokens bouncing back again.

1. First you must install the Gravity Bridge command line interface (CLI) tool called `gbt` from the official Gravity Bridge GitHub. To do this, follow only the "Download and Install CLI tools" instructions on [this page](/docs/cli-usage.md#download-and-install-cli-tools) to install the `gbt` tool, it is not necessary to install the `gravity` tool. Stop when you get to `Sending ERC20 tokens from Ethereum to Gravity Bridge` or any further instructions.

1. Double check that the tool you have installed is from the official Gravity Bridge source code [GitHub](https://github.com/Gravity-Bridge/Gravity-Bridge). If someone is trying to get you to install a tool from another website, or using instructions not on the official Gravity Bridge documentation [GitHub](https://github.com/Gravity-Bridge/Gravity-Docs/), stop immediately and ask for help in the [Discord](https://discord.gg/b3DtsNPu). Scammers are very persistent, be certain you are working with only the official tools and instructions!

1. Obtain your Ethereum private key used for the `canto1abcdefg4567...` account. This is probably the private key for your Ethereum account in MetaMask. **Be very careful during this process! Do NOT share your private key with ANYONE, and make sure that you delete your private key after this process. If someone gains control of your private key, your entire Ethereum AND Canto accounts are vulnerable! You have been warned!** You can use the [Metamask Instructions here](https://metamask.zendesk.com/hc/en-us/articles/360015289632-How-to-export-an-account-s-private-key) to obtain the private key for use with `gbt`.

1. Figure out what your total amount to be transferred is. This must be provided without any decimal places, in the case of WETH and other 18 decimal ERC20 tokens, 1 WETH would be `1 * 10^18 = 1000000000000000000`. USDC is a 6 decimal token so 0.22 USDC would be `0.22 * 10^6 = 220000`. We recommend checking out the token page on Etherscan for your ERC20 which lists the token decimals, like [this page for USDC.](https://etherscan.io/token/0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48)

1. Figure out the "denom" you will be transferring. For an ERC20 this is going to be the name of the gravity voucher for that ERC20, which is "gravity" + the **properly capitalized** ERC20 contract address. USDC is `gravity0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` and WETH is `gravity0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2`.

1. Now that you have the `gbt` CLI tool installed you have two recovery options:

    - **Send your tokens back to Ethereum** and try again (Best for getting back to a clean slate)
    - **Send your tokens to a more easily controlled Gravity Bridge account** (Cheaper, best for continuing what you wanted to do)

### Send your tokens back to Ethereum

1. Choose a bridge fee amount. This will vary depending on the token you are trying to send to Ethereum, but you will need to provide a reasonable fee to get a reasonable bridge time. The standard recommendation is $10USD worth of the token, but if you would like the tokens to arrive faster you can pay more to make that happen. You might be able to check what [Blockscape](https://bridge.blockscape.network) recommends for a Gravity Bridge -> Ethereum transfer and choose one of those values. Your fee amount will also need to be provided in the same fashion as the amount in step 5 above.

1. Fill in your specific values in to the following command, and make sure there is no space between \<Transfer Amount\> and \<Denom\> or between \<Bridge Fee Amount\> and \<Denom\>, e.g. `123456789gravity0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`. **If you are trying to send all of the balance in your Gravity account, you must subtract the bridge fee amount from the total amount you discovered in step 5 above to arrive at your transfer amount.**

`gbt keys recover-funds --cosmos-grpc http://gravitychain.io:9090 --send-to-eth --ethereum-key <Your Ethereum Private Key> --amount <Transfer Amount><Denom> --eth-bridge-fee <Bridge Fee Amount><Denom> --eth-destination <Your Ethereum Address>`

Finally, run your command and wait for it to appear in the Transaction Queue on this [Gravity metrics website](https://info.gravitychain.io/).
**Make sure that you remove all traces of your Ethereum private key, including in your clipboard and the terminal used to run the command.**

If you do not see your funds in a reasonable timeframe, ask for help on the [Discord](https://discord.gg/b3DtsNPu).

### Send your tokens to a more easily controlled Gravity Bridge account

1. Determine your Gravity account destination. Generate a new Gravity Bridge account, or use an existing one you already control. We recommend using [Keplr](https://www.keplr.app/) to make this account since it will work with the Blockscape website later.

1. Fill in your specific values in to the following command, and make sure there is no space between \<Transfer Amount\> and \<Denom\>, e.g. `123456789gravity0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`.

`gbt keys recover-funds --send-on-cosmos --ethereum-key <Your Ethereum Private Key> --amount <Transfer Amount><Denom> --cosmos-destination <Gravity account destination>`

The balance should appear in Keplr within a minute, after which you can use the [Blockscape Website](https://bridge.blockscape.network)
to continue what you were doing earlier, even sending the tokens back to Ethereum if you wish. If the tokens don't arrive in
Keplr, reach out for help on the [Discord](https://discord.gg/b3DtsNPu).