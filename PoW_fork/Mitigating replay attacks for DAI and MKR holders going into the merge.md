# *Brace for impact!* Mitigating replay attacks for DAI and MKR holders going into the merge.

*The following guide provides context and information about replay attacks, a potential proof of work Ethereum fork with a chainId of 1 and potential effects on MakerDAO contracts, as well as risks of using DAI's permit function on all Ethereum mainnet forks. All of the following information does not constitute advice of any form.*

The merge is upon us! Finally, after years of extensive testing and work, the transition proof of stake Ethereum is ready. However, in the last few months, miners along with other parties have raised the possibility of a hard fork of Ethereum, creating a proof of work chain (PoW) as well as a proof of stake chain (PoS).

## What's the consensus?

In general, most protocols and companies are opting for support of the PoS chain, with kingmakers such as [USDC issuer Circle](https://www.circle.com/blog/usdc-and-ethereums-upcoming-merge) , [oracle provider ChainLink](https://docs.chain.link/docs/ethereum-proof-of-stake-merge/) and [WBTC custodian BitGo](https://blog.bitgo.com/bitgos-approach-to-the-ethereum-merge-60ef71a9c095?gi=718f898c3521) already announcing support for the PoS chain. BitGo's announcement states the following:

>BitGo protects all transactions originating from our platform against replay attacks; for example if you have a confirmed transaction on the PoS chain (which will maintain the current chain ID), then that transaction cannot be “replayed” on a given forked chain, as the latter **will necessarily have a different chain ID**.

Replay attacks? Those sound scary! However, we can be certain that the PoW ETH will change `chainId` to allow for as smooth a hard fork as possible, right? Right?

![[Tweet 1.png]]
*[Oh.](https://mobile.twitter.com/LefterisJP/status/1567077681786769409) Maybe they just weren't made aware and needed a reminder!*

![[PoW_PRs.png]]
[*Oh no, this isn't good...*](https://github.com/ethereumpow/go-ethereum/pulls)

Based on this, it looks like there's a non-zero (and in my personal view, very high) chance that PoW ETH *will use the same chainId* as PoS ETH. This is **really not good** if you don't know what you're doing. The following discussion assumes that the `chainId` of the PoW fork will be `1`. Let's discuss why this is *really* bad and how you can avoid being impacted.

## What is a replay attack?

A replay attack is taking a transaction from one network and using it on another network. Let's say you sent someone 100 wETH on the PoW fork; you wouldn't want this action to be repeatable on the Ethereum mainnet without your permission, right?

> *A replay attack is the broadcasting of a transaction from one blockchain/network on another blockchain/network.*

A replay attack involves taking your transaction on PoW ETH and **replaying** it on PoS ETH. [EIP-155](https://github.com/ethereum/EIPs/issues/155) was implemented to prevent replay attacks by using a unique `chainId` value for different networks (such as Binance Smart Chain, Polygon and testnets like Rinkeby and Goerli) **however if the same `chainId` is used on two networks, replay attacks become possible again.**

## How do replay attacks affect DAI and MKR owners?

Replay attacks are not specific to DAI or MKR tokens; in fact they're not specific to anything at all! Raw ether, ERC20 tokens, ERC721 NFTs and all other digital assets can be transferred through replay attacks. **Any transaction broadcast to the PoW network can be replayed on the PoS network if they share the same chainId.** 

Some examples:
- You choose to interact with any Maker contracts on PoW directly (from the same address you used before the fork) --> This transaction can be replayed on PoS, any interactions you made may be replicated on PoS.
- You choose to sell PoW DAI or MKR tokens to someone in a peer-to-peer transaction --> This transaction can be replayed, the transfer can be replicated on PoS.
- You choose to buy or sell PoW DAI or MKR tokens on an exchange such as Uniswap --> This transaction can be replayed, the interaction may be replicated on PoS.

The long and short of it here is that **anything you do on PoW ETH *may* be able to be replayed on PoS ETH and you should assume that it can be replayed on PoS.** My usage of the word 'may' here is because some replay attacks will revert on one network but not another, however they [still constitute attacks in my view.](https://twitter.com/0xArbiter/status/1567143219237904384)


## How do replay attacks affect Maker Protocol beyond this?

Clearly this is a pretty big issue to address, from a technical standpoint. It's also worth addressing that the very idea of forking Ethereum mainnet *while maintaining a `chainId` of 1* is considered insane. Here are the main elements of the DSS system which will cause issues:

##### Oracles

As discussed earlier, ChainLink has already sided with PoS over PoW. This means that the Maker Protocol [oracle module](https://docs.makerdao.com/smart-contract-modules/oracle-module) will break instantly in absence of oracle price feeds being forwarded to it. Additionally, the [median.sol](https://github.com/makerdao/median/blob/master/src/median.sol) contract does not have any verification of `chainId` value, meaning that *PoS asset prices* can be used (via replay attacks) to "force" the MCD system to "work". While this would keep the price of DAI at $1 in theory, other elements of MCD would break and DAI would become massively over-minted. 

*`Median` can be used to force DSS to keep operating on PoW based on PoS price feeds, however the true collateralisation ratio will be much, much lower than target.*

##### DAI token

The [DAI token contract](https://github.com/makerdao/dss/blob/master/src/dai.sol) is immutable and was provided a `chainId` in it's constructor. The DAI  `permit` function does not check `chainId`. This makes interactions with DAI unsafe on any 'proper' fork (one which changes the `chainId`) of Ethereum `mainnet`.  Again, regardless of whether the `chainId` is updated on the fork, usage of DAI on any fork of mainnet could result in loss of DAI on mainnet via replay attack.  

*`Permit` can be used to transfer users' funds via replay attack on any forks of Ethereum mainnet (not only those with a `chainId` of 1).*

##### What does this mean for DAI and MKR holders?

In my view "you do not need to do anything for the upcoming merge!" should read as "you *need to not do anything* for the upcoming merge!".  Here are your options for PoW chain interaction, in order of increasing danger:

1) Do not interact with the PoW fork at all (no interactions at all).
2) Do not interact MKR, DAI or any MakerDAO associated contracts on the PoW fork.
3) Interact with the fork, taking extreme precautionary measures and **assume that all of your transactions will be replayed on mainnet.** Avoid the use of `permit` and do not assume that DAI will be using the correct price feeds or that it is sufficiently collateralised.

Additionally:

>**For reasons above, it is highly recommended that, unless you know what you're doing, you do NOT interact with DAI on any `mainnet` fork of ethereum (regardless of `chainId`).** 

My personal choice here would be to not interact with the PoW fork at all unless you are sufficiently technical and know what you're doing.

## Wrapping up

If the PoW fork backers decide to use a `chainId` of 1, the PoW fork will be incredibly dangerous for all users, including technical and experienced ones. [Bitcoin Cash](https://www.circle.com/blog/preventing-replay-attacks-after-the-bch-hard-fork) and [Ethereum Classic](https://www.coindesk.com/markets/2016/07/29/rise-of-replay-attacks-intensifies-ethereum-divide/) experienced replay attacks (even with ETC using a `chainId` of 61) which claimed the funds of new and old investors alike. 

The decision to interact with the PoW fork is yours to make - hopefully you are more educated on some of the (in my opinion, very large) risks present if you do so.