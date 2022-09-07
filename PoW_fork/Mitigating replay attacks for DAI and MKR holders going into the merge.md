# *Brace for impact!* Mitigating replay attacks for DAI and MKR holders going into the merge.

*The following guide provides context and information about replay attacks, a potential Ethereum hard fork and potential effects on MakerDAO contracts, as well as risks of using DAI's permit function on all Ethereum mainnet forks. None of the following information constitutes advice of any form.*

The merge is upon us! Finally, after years of extensive testing and work, the transition proof of stake Ethereum is ready. However, in the last few months, miners along with other parties have raised the possibility of a hard fork of Ethereum, creating a proof of work chain (PoW) as well as a proof of stake chain (PoS).

## What's the consensus?

In general, most protocols and companies are opting for support of the PoS chain, with kingmakers such as [USDC issuer Circle](https://www.circle.com/blog/usdc-and-ethereums-upcoming-merge) , [oracle provider ChainLink](https://docs.chain.link/docs/ethereum-proof-of-stake-merge/) and [WBTC custodian BitGo](https://blog.bitgo.com/bitgos-approach-to-the-ethereum-merge-60ef71a9c095?gi=718f898c3521) already announcing support for the PoS chain. BitGo's announcement states the following:

>BitGo protects all transactions originating from our platform against replay attacks; for example if you have a confirmed transaction on the PoS chain (which will maintain the current chain ID), then that transaction cannot be “replayed” on a given forked chain, as the latter **will necessarily have a different chain ID**.

Replay attacks? Those sound scary! However, we can be certain that the PoW ETH will change `chainId` to allow for as smooth a hard fork as possible, right? Right?

![[Tweet 1.png]]
*[This issue has been raised already.](https://mobile.twitter.com/LefterisJP/status/1567077681786769409) 

![[PoW_PRs.png]]
[*Open pull requests changing the chain ID remain unmerged.*](https://github.com/ethereumpow/go-ethereum/pulls)

Based on this, it appears that there's a non-zero chance that PoW ETH *will use the same chainId* as PoS ETH. The following discussion assumes that the `chainId` of the PoW fork will be `1`. Let's discuss why this is *really* bad and how you can minimise the risk that you are affected by this.

## What is a replay attack?

A replay attack is taking a transaction from one network and using it on another network. If you sent someone 100 wETH on the PoW fork; you wouldn't want this action to be repeatable on the Ethereum mainnet without your permission, however replay attacks make this possible.

> *A replay attack is the broadcasting of a transaction from one blockchain/network on another blockchain/network.*

A replay attack involves taking a transaction on PoW ETH and **replaying** it on PoS ETH. [EIP-155](https://github.com/ethereum/EIPs/issues/155) was implemented to prevent replay attacks by using a unique `chainId` value for different networks (such as Binance Smart Chain and Polygon as well as testnets like Rinkeby and Goerli) **however if the same `chainId` is used on two networks, replay attacks become possible again.**

## How do replay attacks affect DAI and MKR owners?

Replay attacks are not specific to DAI or MKR tokens; in fact they're not specific to anything at all! Raw ether, ERC20 tokens, ERC721 NFTs and all other digital assets can be transferred through replay attacks. **Any transaction broadcast to the PoW network can be replayed on the PoS network if they share the same chainId.** 

Some examples of interactions which may be replayable:
- Interacting with any Maker contracts on PoW directly (from the same address you used before the fork)
- Selling PoW DAI or MKR tokens to someone in a peer-to-peer transaction
- Interacting with PoW DAI or MKR tokens via a dApp (such as selling them on UniSwap)

The long and short of it here is that **anything you do on PoW ETH *may* be able to be replayed on PoS ETH and you should assume that it can be replayed on PoS.** 

## How do replay attacks affect Maker Protocol beyond this?

Clearly this is a pretty big issue to address, from a technical standpoint. Here are the main elements of the DSS system which will cause issues:

##### Oracles

 The [median.sol](https://github.com/makerdao/median/blob/master/src/median.sol) contract (part of the  [oracle module](https://docs.makerdao.com/smart-contract-modules/oracle-module)) does not have any verification of `chainId` value, meaning that *PoS asset prices* can be used (via replay attacks) to "force" the MCD system to "work". While this would keep the price of DAI at $1 in theory, forcing PoS oracle values into PoW MCD would likely overvalue the underlying collateral, meaning that DAI on PoW would be backed by less value in reality than the system would think. **At this point, undercollateralized PoW DAI is not expected to impact PoS DAI.**

*`Median` can be used to force DSS to keep operating on PoW based on PoS price feeds, however the true collateralisation ratio will be much, much lower than target.*

##### DAI token

The [DAI token contract](https://github.com/makerdao/dss/blob/master/src/dai.sol) is immutable and was provided a `chainId` in it's constructor. The DAI  `permit` derives the digest from the `DOMAIN_SEPARATOR` which is based on the  `chainId` provided in the constructor rather than the current `chainId`. This makes interactions with DAI unsafe on any 'proper' fork (one which changes the `chainId`) of Ethereum `mainnet`.  Again, regardless of whether the `chainId` is updated on the fork, usage of DAI on any fork of mainnet could result in loss of DAI on mainnet via replay attack.  

*The use of `permit` on any hard fork of Ethereum's current state could be used to transfer users' funds via a replay attack on other forks, including `mainnet`.*

##### What does this mean for DAI and MKR holders?

Here are some options for PoW chain interaction, from safest to riskiest:
1) Do not interact with the PoW fork at all (no interactions at all).
2) Do not interact MKR, DAI or any MakerDAO associated contracts on the PoW fork.
3) Interact with the fork, taking extreme precautionary measures and **assume that all of your transactions will be replayed on mainnet.** Avoid the use of `permit` and do not assume that DAI will be using the correct price feeds or that it is sufficiently collateralised.


## Wrapping up

If the PoW fork backers decide to use a `chainId` of 1, the PoW fork will be incredibly dangerous for all users, including technical and experienced ones. [Bitcoin Cash](https://www.circle.com/blog/preventing-replay-attacks-after-the-bch-hard-fork) and [Ethereum Classic](https://www.coindesk.com/markets/2016/07/29/rise-of-replay-attacks-intensifies-ethereum-divide/) experienced replay attacks (even with ETC using a `chainId` of 61) which claimed the funds of new and old investors alike. 

 It is worth nothing that interacting with the Maker Protocol on **any future fork of Ethereum mainnet's current state** is dangerous - **even if the fork uses a different `chainId` value** - due to the aforementioned issues.