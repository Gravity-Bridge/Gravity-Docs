# Bridging Assets to EVMOS

Gravity Bridge supports moving assets between Ethereum and EVMOS. This guide will demonstrate how to do
so.

First visit [bridge.blockscape.network](https://bridge.blockscape.network) and select Evmos from the destination list.
![image](https://user-images.githubusercontent.com/19688153/182438709-e9954667-82bf-4b3c-8175-ad9b015675eb.png)

Connect your MetaMask wallet and your Keplr wallet. Alternately you can skip keplr and click the lock button next to transfer address and paste your evmos1 address there.

Click select token and select which token you would like to bridge and the amount.

![image](https://user-images.githubusercontent.com/19688153/182438867-acb8a81d-a284-43b3-89e0-841ece06c604.png)

Once ready click the 'begin transfer' button and sign with Metamask. About 10 minutes later your tokens will appear in "IBC Balance" column on [Evmos](https://app.evmos.org/assets)

From there you can follow [the EVMOS assets guide](https://medium.com/evmos/assets-page-quick-reference-guide-f1cd09e7cb85) to convert your Gravity IBC tokens to ERC20 representations for use on EVMOS!

## Moving assets back to Ethereum

Following the [the EVMOS assets guide](https://medium.com/evmos/assets-page-quick-reference-guide-f1cd09e7cb85) on [Evmos](https://app.evmos.org/assets) you must first use the convert button to unwrap your tokens.

Once unwrapped you can use [bridge.blockscape.network](https://bridge.blockscape.network) to bring your unwrapped tokens back to Gravity Bridge via IBC.

Select Evmos as your source chain and Gravity Bridge as the destination, connect Keplr, select the token amount, and sign the transfer.

![image](https://user-images.githubusercontent.com/19688153/182439921-2bff196f-23ce-4bbf-8477-c2058e501326.png)

Now you can take your tokens from Gravity Bridge to Ethereum, select Gravity Bridge as the source and Ethereum as the destination and input an Ethereum destination address
either via Metamask or unlocking the destination box and pasting a destination. Sending directly to exchanges as a destination address is supported.

