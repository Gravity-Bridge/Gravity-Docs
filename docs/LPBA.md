# **Bridging Tokens and using Gravity Bridge with Osmosis**

This is an introduction to using Gravity Bridge to bridge Ethereum ERC20 tokens into the Cosmos ecosystem, and how to use them on OSMOSIS DEX.

Before proceeding, you’ll need to take the following preliminary steps:

- Install both Keplr and Metamask wallet browser extensions in the browser you will be using
- Add Gravity to your Keplr wallet (visit the [Gravity Bridge Commonwealth forum](https://commonwealth.im/gravity-bridge) and connect your wallet, this will add it to Keplr)
- Have USDC, wETH, or DAI in your Metamask wallet (to be bridged)
- Have ETH in your Metamask wallet to pay gas fees

## Step 1: Gravity Bridge front-end

There are currently two independent Gravity Bridge front-ends, with more on the way.

[SpaceStation.zone](https://spacestation.zone/) (maintained by [Cosmostation](https://www.cosmostation.io/))
[Gravity Bridge Portal](https://bridge.blockscape.network/) (maintained by [Blockscape](https://blockscape.network/))

In this tutorial we will be using [spacestation.zone](https://spacestation.zone/). The first step is to connect your Metamask and Keplr wallets, by clicking on the linking respective buttons.

![Image 01](https://i.imgur.com/oafLhD7.png)
After you have done so, use the drop-down menu to select a token that you would like to Bridge into the Cosmos ecosystem.
![Image 02](https://i.imgur.com/saoPlZT.png)For the purposes of this tutorial, we will be bridging USDC. Enter the amount of USDC that you would like to bridge into your Gravity wallet and click “transfer”.
![Image 03](https://i.imgur.com/vT9zBPr.png)
A pop-up will appear with a preliminary transaction review window. Click transfer again to start the transaction.
![Image 04](https://i.imgur.com/dLXXDRg.png)At this point, review the transaction, including gas fees. Note that you will need to have sufficient Ethereum in your wallet to cover them, and the gas costs will vary throughout the day. After reviewing the transaction, click “confirm”. The freshly minted USDC IBC tokens will be deposited in your Gravity wallet. To view them, we’ll use the [Skynet block explorer](https://www.skynetexplorers.com/gravity-bridge).
![Image 05](https://i.imgur.com/aMWAajb.png)
The next step is to send the tokens that you’ve just bridged into your Gravity wallet, over to your Osmosis wallet. To do so, use the small arrow in the middle of the screen to change the source and destination.
![Image 06](https://i.imgur.com/b9RLW5w.png)
Clicking on the destination tab will bring up the drop-down menu, which you can use to select a new destination. Select Osmosis. This will connect your wallet again, this time to the destination side (as well as the source side) of the transfer window.
![Image 07](https://i.imgur.com/HjAjdb2.png)![Image 08](https://i.imgur.com/svwD7ni.png)You can now input the amount of USDC you’d like to send to your Osmosis wallet and send them directly there. After the transfer you should inspect your Osmosis wallet in Keplr and see the bridged tokens waiting for you, although they might appear looking like this:
![Image 09](https://i.imgur.com/lYZ3jbB.png)
Until Keplr completes full support of Gravity Bridge, bridged tokens may be displayed with the GRAVITY prefix followed by the contract address of the ERC20 token you have bridged. On a similar note, because Gravity Bridge is in an Alpha stage on Osmosis, your bridged tokens may not yet appear in the [assets page of the Osmosis front end](https://app.osmosis.zone/assets). Not to worry! You can still fully utilize Osmosis liquidity pools to swap and pool your bridged assets.

To do so, navigate to [Osmosis](https://app.osmosis.zone/), and click on POOLS from the menu on the left. You can then scroll to the very bottom, and manually scroll through the list of “all pools” to find the appropriate liquidity pool. In this case, we’ll be using LP #633, which comprises USDC + Osmo. To save you some time, you can use this direct link: [https://app.osmosis.zone/pool/633](https://app.osmosis.zone/pool/633)

From here you can add/remove liquidity to the liquid pool or swap tokens. Here we’ll focus on swapping tokens. Click on the Swap token button and adjust the browser so to have a better overview of the different fields.

![Image Other](https://i.imgur.com/TRgCqlK.png)
Note that the token balance does not display in the native token. In this case, it is displaying 18 USDC as 17,485,362 gravity0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48. I know this, because in this instance I have just sent 14 USDC to my Osmosis address. If I click “MAX” I am able to trade the 14 USDC for OSMO (with a roughly equivalent dollar value).

It is easiest to use the “HALF/MAX” button to wake up the input field, and then you are free to do some simple math and enter a 8-digit number that corresponds to the amount of token that you would like to swap into Osmo.

![Image 10](https://i.imgur.com/RQ0KBNd.png)
If you’d like to exchange your Osmo for additional USDC, you can simply click on the center button to reverse the position of the tokens in the exchange window, enter the amount of Osmo you’d like to exchange for USDC, and click "swap."
