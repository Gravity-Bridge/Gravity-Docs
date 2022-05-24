# Relaying for Gravity Bridge

The Gravity Bridge module has two major data flows, one from Ethereum to Gravity Bridge and one from Gravity Bridge to Ethereum.

Every validator runs the `gbt orchestrator` command which provides the Ethereum -> Gravity Bridge flow. Relayers
provide the Gravity Bridge -> Ethereum data flow by reading data from the Gravity Bridge module and submitting it as Ethereum transactions.

To understand more see the [relaying rewards](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/docs/design/mint-lock.md#relaying-rewards) of the Gravity Bridge documentation.

## Download and install the relayer

### Linux

```bash

mkdir gravity-bin
cd gravity-bin
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.2/gbt
chmod +x *
sudo mv * /usr/bin/

```

### Mac

```bash

mkdir gravity-bin
cd gravity-bin
wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.2/gbt-mac-amd64
chmod +x *

```

### Windows

Download [gbt](https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.5.2/gbt.exe)

You will need to open a terminal and nagivate to the directory where these files where downloaded before running any commands.

## Risks

Relaying for Gravity Bridge is not risk free. Only put as much ETH into your relayer as you are willing to lose.

The fundamental challenge of relaying is that all relayers are competing to submit the same transactions to get the reward. Because it's impossible to know in advance with transactions an Ethereum miner will decide to include in a block there is no way to avoid the risk of your TX failing because someone else claimed the relaying reward first. Unfortunately due to the design of Ethereum relayers must pay for their failed transactions and since the batch arguments are quite large the failed tx will have a gas cost on the order of 100k gas.

The way to manage this risk is to reduce the probability that you and another relayer both submit the same transaction batch or valset at the same time. The best way to do this would involve some advanced mempool inspection and history inspection to determine how many other relayers are active. This becomes extremely complicated for batches, where relayers may have a specific interest in different assets, so they each have to be tracked distinctly.

Right now there is only one knob to tune. After running `gbt init` you can edit `~/.gbt/config.toml` and modify the `relayer_loop_speed`.

```text
[relayer]
relayer_loop_speed = 600
```

We can do some math to show how `relayer_loop_speed` translates to expected collisions. Assuming a constant 15 second block period for Ethereum.

```text
    expected number of collisions = (num relayers - 1 ) * (block time / relayer_loop_speed)

    For a 17 second loop time and two relayers we expect .88 collisions

    For a 600 second loop and two relayers we expect .025 collisions

    For a 600 second loop and one hundred relayers we expect 2.5 collisions
```

Hopefully this makes it clear that setting the `relayer_loop_speed` to a lower value presents a lower risk, but also a lower chance of receiving a reward.

## Rewards

There are two primary classes of relaying rewards. One are batch fees, which are always denominated in the same token type as the batch is relaying and the other is Validator set update rewards, which will issue Graviton once the community votes to enable it.

You can read more about [relaying rewards](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/docs/design/mint-lock.md#relaying-rewards) section of the Gravity docs.

## Running a relayer

In order to relay you must provide an Ethereum address containing some funds.

Your relayer can optionally automatically request batches (requires sending a tx on the Gravity Bridge chain) all you need is the private key phrase containing `1ugraviton` to initialize the account since there is no min fee. Add the options `--cosmos-phrase "your private key phrase containing graviton here"` and `--fees 0ugraviton` if you would like to do this.

```bash
gbt init
gbt relayer \
--ethereum-key 0xYOURKEYHERE
--cosmos-grpc https://gravitychain.io:9090
--ethereum-rpc https://eth.althea.net
--gravity-contract-address "0xa4108aA1Ec4967F8b52220a4f7e94A8201F2D906" \
```

## Relaying a specific token

Please see the [reference config](https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/orchestrator/gbt/src/default-config.toml).

By placing the following config block into `~/.gbt/config` you can configure your relayer to relay all profitable batches at the provided profit margin. With profitable being defined
as the price in Uniswap vs ETH spent to relay the batch. You can also add specific whitelisted tokens where you bypass the oracle and set a price (in USDC per whole token) manually.

For both these parameters it is possible to set low values. For example an `amount` of zero in the whitelist will relay any batch of that token type without regard for reward. Likewise a `margin` value less than `1.0` will relay batches that only partially cover the cost of relaying.

```text
# Use whitelist mode to relay tokens which may not be listed
# in Uniswap, or if you want to relay a specific type for free
# you can set the amount to "0"
#
[relayer.batch_relaying_mode]
mode = "ProfitableWithWhitelist"
margin = 1.5
[[relayer.batch_relaying_mode.whitelist]]
token = "0x6Bd41fCdF129297c3524395d669c0865b3CA85B2"
price = "10"
decimals = 18
```
