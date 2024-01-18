# What is redemption

Any btUSD holder could always redeem X btUSD against the system for $X worth of collateral. However, there would be a dynamic fee applied to redemption, corresponding to the redemption size related to the total supply of btUSD: the more btUSD redeemed, the more fee charged.

For example, if the current redemption fee is 1% and the collateral price is $50000, then you redeem 1000 btUSD, you would get 0.0198 collateral (0.02 minus a redemption fee of 0.0002).

All active Troves in the system will take a "socialized" share when redemption happens. This means each Trove would expect a decrease in both collateral and debt but with a higher(healthier) ICR after redemption. 

Note if the system TCR is lower than MCR, then redemption is not allowed until TCR return to a higher level.

## Why redemption is needed?

Redemption enforce a low pricing bound for btUSD, i.e., when btUSD is traded **below** $1 peg, redemption incentivizes arbitrageur to help fixing the peg. 

For example, if btUSD is traded at $0.98 in the market, then arbitrageur could buy 100 btUSD from market using $98  then redeem it to receive $99 worth of collateral (considering 1% redemption fee), thus resulting a $1 net profit for arbitrageur. Since less btUSD exist in the market after redemption, the price peg is expected to restore efficiently.

## As a Trove owner, do I expect a loss if redemption happens? 
If redemption happens, troves do not suffer any loss in face value since the redemption is executed at exactly the reported collateral price. However, a trove will lose part of its original collateral exposure.  

## Math for redemption 

Following are some math proof provided for interested users. Feel free to skip this section.

### M0: Redemption fee calculation and decaying

The redemption fee is given by the formula $(baseRate + 0.5\%) * CollateralWorthOfRedeemed$
 
The `baseRate` will increase by each redemption and decay to 0 over time at a half-life rate of 12 hours. Specifically, upon each redemption:

* `baseRate` is decayed based on how many minutes are passed since the last fee event (either borrowing or redemption)
* `baseRate` is incremented by an amount proportional to the fraction of the total btUSD supply that was redeemed
* $baseRateNew = baseRateOld + \frac{btUSDRedeemed}{2 * btUSDTotalSupply}$

Where `baseRateOld` is the value just prior to this redemption, and `baseRateNew` is the new value (and gets applied to this redemption).

### M1: ICR will increase after redemption

Suppose a trove (not liquidatable) with original collateral $C$ and debt $D$ when the collateral price is $p$, now a redemption happens and bring socialized reduction to trove's collateral and debt: $\Delta{c}$ and $\Delta{d}$ respectively. Then we could compare the trove's ICR before and after redemption:

- Before redemption, $ICR_{before} = \frac{C * p}{D} \gt 110\%$
- After redemption, $ICR_{after} = \frac{(C - \Delta{c}) * p}{D - \Delta{d}} = \frac{(C - \Delta{c}) * p}{D - \Delta{c} * p}$
- The trove's ICR would increase: $ICR_{after} - ICR_{before} = \frac{(C - \Delta{c}) * p}{D - \Delta{c} * p} - \frac{C * p}{D} = \frac{\Delta{c}*p*(C*p - D)}{(D - \Delta{c} * p)*D} \gt 0$

### M2: ICR relation between existing Troves will remain the same

Suppose two existing troves (not liquidatable) with same original collateral $C$ but different debts $D_{1} < D_{2}$ when the collateral price is $p$, now a redemption happens and bring socialized reduction to both two trove's collaterals and debts: $\Delta{c}$ and $\Delta{d}$ respectively. Then we could compare these two trove's ICRs after redemption:

- Before redemption, $ICR_{1Before} = \frac{C * p}{D_{1}} \gt ICR_{2Before} = \frac{C * p}{D_{2}} \gt 110\%$
- After redemption, $ICR_{1After} = \frac{(C - \Delta{c}) * p}{D_{1} - \Delta{d}}$
- After redemption, $ICR_{2After} = \frac{(C - \Delta{c}) * p}{D_{2} - \Delta{d}}$
- The two trove's ICRs ratio would be: $\frac{ICR_{1After}}{ICR_{2After}} = \frac{D_{2} - \Delta{d}}{D_{1} - \Delta{d}} \gt \frac{D_{2}}{D_{1}}$

### M3: ICR relation between new and old Troves will remain the same

Suppose an existing trove A (not liquidatable) with original collateral $C$, debt $D$ and stake $S_{A} = C * \frac{S_{snapshot}}{C_{snapshot}}$ when the collateral price is $p$, now a redemption of worth-collateral size $c_{R1}$ happens and bring socialized reduction to the trove's collateral and debt: $\Delta{c_{A1}}$ and $\Delta{d_{A1}}$ respectively. 

- After this redemption, total collateral snapshot of the system decrease: $C_{snapshot} = C_{snapshot} - c_{R1}$
- After this redemption, trove A's ICR: $ICR_{A} = \frac{(C - \Delta{c_{A1}})* p}{D - \Delta{d_{A1}}}$

Now another trove B is created with same collateral $C$ and debt $D$ but its stake is calcualted as $S_{B} = C * \frac{S_{snapshot}}{C_{snapshot} - c_{R1}} \gt S_{A}$, meaning it will get more share reduction for future redemption than trove A. At this moment, trove B's $ICR_{B} = \frac{C*p}{D}$ is less than $ICR_{A}$

Some time passes. Then another redemption of worth-collateral size $c_{R2}$ happens and brings socialized reduction to trove A's collateral and debt: $\Delta{c_{A2}}$ and $\Delta{d_{A2}}$ respectively. Note $c_{R2} \lt (C_{snapshot} - c_{R1})$ since you can't redeem all collateral in the system.

Considering how redemption share is calculated with respect to trove' stake, we could get trove B's reduction share for this redemption: $\Delta{c_{A2}} * \frac{S_{B}}{S_{A}}$ and $\Delta{d_{A2}} * \frac{S_{B}}{S_{A}}$ respectively. So now we could compare both troves' ICR:

- trove A's $ICR_{A2} = \frac{(C - \Delta{c_{A1}} - \Delta{c_{A2}})* p}{D - \Delta{d_{A1}} - \Delta{d_{A2}}} = \frac{(C - \Delta{c_{A2}})*p - \Delta_{c_{A1}}*p}{D - \Delta_{c_{A2}}*p- \Delta_{c_{A1}}*p}$
- trove B's $ICR_{B2} = \frac{(C - \Delta{c_{A2}} * \frac{S_{B}}{S_{A}}) * p}{D - \Delta{d_{A2}} * \frac{S_{B}}{S_{A}}}$, we could note $\frac{S_{B}}{S_{A}}$ as $K \gt 1$
- With a bit rearrange, trove B's ICR could be written as $ICR_{B2} = \frac{(C - \Delta{c_{A2}})*p - (K - 1) * \Delta_{c_{A2}}*p}{D - \Delta_{c_{A2}}*p - (K - 1) * \Delta_{c_{A2}}*p}$
- By observing the formula of trove A's and trove B's ICR, the problem now comes down to a question: which is larger? $\Delta_{c_{A1}}$ or $(K - 1) * \Delta_{c_{A2}}$
- $\frac{(K - 1)*\Delta_{c_{A2}}}{\Delta_{c_{A1}}} = (\frac{C_{snapshot}}{C_{snapshot} - c_{R1}}-\frac{C_{snapshot - c_{R1}}}{C_{snapshot - c_{R1}}})*\frac{\Delta{c_{A2}}}{\Delta_{c_{A1}}}=\frac{c_{R1}}{C_{snapshot} - c_{R1}}*\frac{\Delta{c_{A2}}}{\Delta_{c_{A1}}} \le \frac{c_{R1}}{C_{snapshot} - c_{R1}}*\frac{c_{R2}}{c_{R1}} \lt 1$
- Applying similar proof in above M1 section, we could know that $ICR_{A2} > ICR_{B2}$ still holds