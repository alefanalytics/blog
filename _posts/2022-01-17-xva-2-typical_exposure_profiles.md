---
layout: post
title: "XVA series - 2. Typical exposure profiles"
tags: [XVA, CVA, Counterparty credit risk]
comments: false
---

In the previous article, we have introduced some common counterparty credit risk metrics such as the expected exposure (EE) and the potential future exposure (PFE) profiles.
In this one, we present the typical shapes of those profiles for the most common interest rates and foreign exchange derivatives.

* TOC
{:toc}

# Framework

As the aim of this article is to illustrate the EE and PFE profile shapes for some trade types, stylized market data was used and only at-the-money trades were considered. The profile shapes might differ for trades that are deep into or out of the money, or with real market data especially in periods of market stress.

The profiles were generated using a Monte Carlo simulation with 20,000 paths. In terms of models, Garman-Kohlhagen's model was used for FX, with stochastic interest rates evolving according to the Hull-White model.

The sampling dates for the Monte Carlo, called *pricing dates* in the following, are daily for the first week then weekly for the first year, and monthly for the rest of the time frame.

Throughout this article, EE and PFE will respectively denote the expectation and quantile of the discounted exposure under the risk-neutral measure.

# Linear trades

## Fixed and IBOR flows

The discounted exposure of a positive future fixed flow is equal to its discounted value. Because it is a martingale under the risk-neutral measure, the EE profile is a horizontal line.

It can be shown that, when the mean reversion speed and volatility $$\sigma$$ of the short rate are low, the zero-coupon bond volatility is proportional to $$\sqrt{t} (T-t) \sigma$$, where $$T$$ denotes the maturity and $$t$$ the pricing date. So, the volatility will increase and then decrease as the pricing date get closer to the maturity, leading to an increasing and concave PFE shape as shown below:

{% include plots/ee_profiles/fixed_flow.html %}

Unlike the fixed flow, the IBOR flow's value is uncertain, and takes the sign of the IBOR in the future, meaning that it might become negative.
As its discounted value has a constant expectation, with a distribution that widens with time. The EE profile will be increasing because the negative values are floored to zero.

In mathematical terms, the discounted exposure is a submartingale because it is obtained by applying a convex function (the positive part function) to a martingale (the discounted value).

{% include plots/ee_profiles/ibor_flow.html %}

In the plot, we can see two regions where the EE is constant:

- In the beginning: as the IBOR value is initially positive, the EE is constant over the time period where the distribution is not wide enough to cross to the negative domain.
- In the end: after the fixing date, 2022-09-13 in the plot, the IBOR value is known and the flow behaves (conditionally on the fixing date) just like a fixed flow, leading to a horizontal EE profile.

This is made clear in the following plot, showing some sample MV trajectories.

{% include plots/ee_profiles/ibor_flow_sample_paths.html %}

## FRAs and interest rates swaps

A forward rate agreement or FRA is the difference between a fixed flow and an IBOR flow. As seen previously, it has increasing EE and PFE profile.

An interest rates swap with fixed and floating legs sharing the same payment dates (and same frequency) is a basket of FRA contracts. 
So, stacking the FRA profiles gives the swap's typical EE profile shape as a bent staircase that:

- goes up initially due to the uncertainty over future payments,
- and then down towards zero as less and less payments remain.

The following plot illustrates this using 5 1Y FRA contracts maturing in 1Y, 2Y, ..., 5Y.

{% include plots/ee_profiles/fra_to_swap.html %}

In the general case where the payment frequencies are different between the swap's fixed and floating leg, the profile will be a bit more complex.

Let us consider for example payer swaps with the following frequencies:

- 3M-3M swap: 3M for both fixed and floating legs;
- 3M-1Y swap: 1Y for the fixed leg and 3M for the floating leg.

As the IBOR flows are not matched by any fixed flow between consecutive fixed payment dates for the 3M-1Y swap, 
the EE will go below the 3M-3M one (above for receiver swaps) then at the next fixed payment date, the bigger fixed flow for the 3M-1Y swap will compensate the 4 floating flows bringing the two profiles back together.

{% include plots/ee_profiles/same_vs_diff_freq_swaps.html %}

An interesting observation is that the swap's profile reaches its maximum around the date corresponding to one third of its maturity.
To explain this, recall that the floating leg (in the mono-curve case) is equivalent to the zero-coupon bond maturing at the swap's maturity (minus a short term but it's not important here).
This zero-coupon bond is driving the variance of the swap, because the fixed leg zero-coupon bonds are multiplied by a rate, usually $$<< 1$$.
From the fixed leg section, we have established that the variance of the zero-coupon bond evolves according to 
$$\sqrt{t} (T-t) \sigma$$. This function reaches its maximum at $$\frac{T}{3}$$.

## FX forwards

The FX forward contract consists in the exchange of two fixed flows in two different currencies at a predetermined future date. 
When the FX forward is at-the-money, which is often the case at its inception, its initial market value is zero.

Usually, the FX volatility is much larger than the interest rates volatilities. So, as a first order approximation, one can neglect this latter.

The reasoning applied here is similar to the one for the IBOR flows, and the shape of the EE and PFE profiles can be explained quanlitatively by considering the MV of the FX forward to be approximately gaussian. In this case, it can be shown[^1] that the PFE's concavity is in $$\sqrt{t}$$ and rate of increase is in $$t$$.

{% include plots/ee_profiles/fx_forward.html %}

## Cross currency swaps

The cross currency swap consists in two swap legs in different currencies, in addition to the exchange of capital flows in these two currencies at maturity.
So, it can be seen as a basket with interest rates swap, and an FX forward.

The capital exchanges taking place at maturity lead to huge EE and PFE for the cross currency swap.

To reduce the cross currency swap's exposure, notional reset clauses are typically considered, as seen in the previous article. In this case, the cross currency swap is said to be *FX-reset* or *marked-to-market*.
The EE and PFE profiles in this case drop significantly at each reset date as the notional of one of the legs is updated to bring the swap back close to the money.

{% include plots/ee_profiles/xccy_swaps.html %}

# Optional trades

## FX options

The seller of an FX vanilla option has no exposure, except if the premium is paid in the future, but we do not consider this case here.
For the buyer, on the other hand, the market value is always positive and the EE is hence equal to the expected discounted market value which is a martingale under the risk neutral measure. As a result, the EE is a horizontal line.

The PFE on the other hand is increasing and usually concave.

{% include plots/ee_profiles/fx_options.html %}

The shapes are similar for calls and puts, and thanks to the call/put parity, the FX forward profiles can be recovered by considering the strategy consisting in buying the call and selling the put.

{% include plots/ee_profiles/fx_parity.html %}

## Caps and floors

Caps (resp. floors) are astrips of multiple IBOR caplets or calls on IBOR (resp. floorlets or puts on IBOR) with different expiries (e.g. every 3 months).
Just like an FX option, each caplet or floorlet will have a horizontal EE profile, and concave increasing PFE profile. This leads to the following staircase shaped EE and PFE profiles for caps and floors.

Here also, thanks to the cap/floor parity, the payer swap's profiles can be recovered by considering a strategy where the cap is bought and the floor sold (or the opposite strategy for a receiver swap).

{% include plots/ee_profiles/cap_floor_swap.html %}

## Swaptions 

A swaption is an option to:

- enter into an interest rates swap at exercise date, in this case the swaption is said to involve a *physical delivery*.
- or to receive the MV of the interest rates swap at exercise date, in this case the swaption is said to involve a *cash delivery*.

{% include plots/ee_profiles/swaptions_vs_fwd_swap.html %}

Just like the other types of options above, before the expiry, the EE profile is horizontal, and the PFE is concave and increasing. 

The cash and physical swaptions are identical in this region. To be more precise, they are near identical due to differences in how the underlying swap's MV is computed in both cases, but these differences are not considered here.

After the expiry, the cash swaption's EE and PFE naturally drop to zero, while those of the physical swaption follow the shape of the swap's ones.

Comparing the physical swaption's EE and PFE profiles with that of the underlying (forward start) swap, we can see that before the exercise date, the swaption's are higher than the swap's, while after the exercise, they are lower.
This is because:

- As the swaption is exercised only on the paths where the swap's MV is positive at the exercise date, its value before is necessarily positive, and higher than the swap's.

- Some of the paths where the swaption was not exercised will actually see the MV of the swap become positive after the exercise date. Those paths will contribute to the swap's EE and PFE, but not to the swaption's.
This leads to a higher EE and PFE for the swap compared to the swaption.

# References
[^1]:
    See for example Appendix 4E of Chapter 11 in: {% include citation.html key="gregory" %}