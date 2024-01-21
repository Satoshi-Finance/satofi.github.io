# How to mint btUSD

## Over-Collateralized Position: Trove

Satoshi Finance utilize a classical [over-collateralization pattern](https://coinmarketcap.com/academy/glossary/over-collateralization) for issuance of the stable coin: btUSD. Each wallet address could have up to one over-collateralized position called "trove" which symbolize its collection of assets within Satoshi Finance: collateral and debt(minted btUSD).

[BTCB](https://www.binance.com/en/collateral-btokens) will be used as the collateral and the minimum collateralization ratio(the dollar value of collateral provided vs the minted amount of btUSD) allowed is as low as 110%, i.e., a user(just any wallet address) could open its trove by providing $1100 worth of BTCB and take out 1000 btUSD. But note that any trove with a collateralization ratio less than 110% will be subject to liquidation, thus keep a reasonable higher collateralization ratio is strongly recommended.

There is also a friendly minimum requirement on how much btUSD a trove should at least mint: 200.

## Any fee would be paid to mint btUSD?
As long as a user mint btUSD from its own trove, an one-time mint fee is charged on the minted amount and included into the trove's total debt. Please note that the mint fee is variable (and determined by algorithm coded in smart contract) and has a minimum floor value of 0.5% in normal scenario. The fee is 0% during Recovery Mode which is triggered when the total collateralization ratio of the entire system reaches 130%. 

No additional interest will be charged so user doesn't need to worry about how long should they keep the trove open. The costs to keep hold of minted btUSD for one week and one year are the same thus make Satoshi Finance a good fit for leveraged bitcoin long trader in the long run. 

The mint fee is calculated by multiplying the amount of minted btUSD with the sum of fee floor value(0.5%) and a "baseRate". For example: If the mint fee stands at 1% and the user wants to mint 1000 btUSD then a fee of 10 btUSD will be charged and the user's trove will have a total debt of 1010 btUSD.
