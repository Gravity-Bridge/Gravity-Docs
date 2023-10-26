# Gravity Bridge Fees and the Auction Module

Gravity Bridge has several inherited fees and a custom fee, making for a confusing landscape. In particular, there are CosmosSDK Tx fees (including Gas), Ethereum gas fees, Send To Ethereum `ChainFee`s, and Send To Ethereum `BridgeFee`s.

## CosmosSDK Tx fees and Gas

Most CosmosSDK blockchains inherit a default fee model including Tx fees and Gas calculations.
These features are commonly used minimally to reduce the risk of persistent spam by making it quite expensive to maintain an attack on the network.
The minimum Tx fee is specified by each individual validator, such that when a Tx is broadcast to any validator the Fee value is checked before being considered for inclusion in the chain.
The Gas calculation provides additional nuance to the Tx fee, allowing the chain to require higher fees to be provided for more difficult operations (e.g. storing data should cost more than some simple calculations), and limiting the execution time of any individual Tx.

## Ethereum Gas fees

The Gravity.sol Solidity contract on Ethereum is subject to Ethereum's Gas system.
With much effort, the Gravity.sol contract is quite efficient in terms of Gas usage as the Ethereum side of the bridge is expected to be much more costly than the Cosmos side.
The Ethereum Gas fees are the reason why Gravity has defined a `BridgeFee`.

## Gravity-specific fees

Gravity has two custom fees paid when users send funds to Ethereum, the `BridgeFee` and the `ChainFee`.

### Bridge Fee

The `BridgeFee` amounts accumulate into transaction batches which are paid out to Relayers for relaying each transaction batch to Ethereum.
There is no minimum value to the `BridgeFee`, but lower values will generally take longer to relay since there is little incentive to do so.

### Chain Fee

The `ChainFee` was introduced with Proposal #86 and has a minimum value determined by the `chain_fee_basis_points` Param.
The `ChainFee` applies only when sending funds to Ethereum and must be a percentage of the amount being sent to Ethereum (not including the `BridgeFee` value), so if `chain_fee_basis_points` is set to 1 and someone sends 10000 USDC to Ethereum, the `ChainFee` provided must be 1 USDC or greater.

### Chain Fee Split

Determined by Proposal #204, `ChainFee` amounts collected by the chain are to be split with 50% of the fee going to use by the auction module, and the other half given to stakers.
This split is determined by the `chain_fee_auction_pool_fraction` value, which is set to "0.5" to match the value decided by Proposal #204.

## Auction Module

The Auction module is a separate CosmosSDK module running on Gravity Bridge Chain alongside the Gravity module.
The function of the Auction module is to facilitate individual auctions for each token held by a special Auction Pool account.

### Auction Lifecycle

The Auction module `auction_length` Param determines the number of blocks between the start and end of the active auctions, called the Auction Period.
At start of the current Auction Period the module will iterate over all balances held by the Auction Pool account and creates an auction for each balance.
The amounts of the active auctions will be removed from the Auction Pool account and held by the Auction Module account.
During the Auction Period, users may submit Bids on the active auctions which involves:

* Specifying the auction to bid on
* Providing a static fee determined by the `min_bid_fee` Param, in GRAV - this will be paid to stakers
* Providing the bid amount, in GRAV - this will be held by the Auction Module account temporarily

When a user bids more on an active auction than the previous bidder, they become the new Highest Bidder for that auction and are set to win the auction at the end of the Auction Period.
The previous Highest Bidder will have their bid amount returned to them, but not the fee they paid.

At the end of the Auction Period all active auctions will be closed.
If an auction received no votes, the auction amount will be sent back to the Auction Pool Account for the next Auction Period.
Otherwise, the auction amount is paid to the Highest Bidder, and the bid amount will be burned if the `burn_winning_bids` Param is true or sent to the Community Pool if false.

Finally, the new auctions will be prepared from the Auction Pool account balances - including any accrued balances during the Auction Period and the recycled balances from auctions with no bidders.

### Auction Params

The Auction module has the following Params which are useful for controlling the module via governance:

* Enabled - If false the active auctions will be stuck in a frozen state where users are unable to bid on them and they will not be paid out at the end of the Auction period. Once this Param becomes true again, the module will close the frozen auctions and start a new period.
* Min Bid Fee - This determines the amount which must be paid in GRAV for each valid Bid on an auction.
* Non Auctionable Tokens - This specifies a blacklist of tokens which will never have active auctions. If the Auction Pool account holds balances of any of these tokens, they will be transferred to the Community Pool instead of having auctions created.
* Burn Winning Bids - This determines if the winning Bid amounts are burned (if true), or sent to the Community Pool (if false).
* Auction Length - This determines the period, in blocks, between the start and end of the active auctions.

### Notes

* Users are not allowed to outbid themselves, so if an account is the current Highest Bidder it may not bid higher until another account outbids.
* Bids are only allowed to be submitted in Grav, as are the fees paid
* The bid fee is separate from the CosmosSDK Tx fee. In fact the Tx fee can be a different token type than GRAV.
* If for some reason the Enabled Param is switched from True -> False -> True before the current Auction Period elapses, the current Auction Period will continue as normal rather than closing.
* Updating the Auction Length param will affect the next auctions created, not the currently active ones.