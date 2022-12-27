---
layout: post
title: "XVA series - 4. Pricing in the CCR/XVA context: Physical-delivery and late-cash options"
tags: [XVA, CVA, Counterparty credit risk]
comments: false
---

As seen in previous articles of this series, the computation of XVA terms and of CCR metrics such as the expected exposure (EE) or potential future exposure (PFE) profiles requires a grid of market values at future dates and under various scenarios.

This leads to some specificities in trade evaluation. This article presents a first example: the treatment of options with late cash or physical delivery. Other examples will be given in subsequent articles, so stay tuned.

* TOC
{:toc}

# Physical-delivery options

An implication of the fact that a market value grid is needed for XVA and CCR computations is that each trade has to be correctly aged along the Monte Carlo simulation dates.

This means that any transformation that the trade undergoes during its life needs to be taken into account. A notable example is options with physical delivery. After it is exercised, such an option will turn into the underlying on the Monte Carlo paths where it is exercised, and will have a market value of zero on the others.

If we are only interested in the market value of the option today, the underlying's value at exercise date $$T$$ is enough to compute it:

$$
MV_\text{option}(0) = \mathbb{E} \left[ D(0,T) \times (MV_\text{underlying}(T))^+ \right] 
$$

On the contrary, as the market value of the option is needed at dates after its exercise date $$t > T$$ for XVA and CCR computations, the fact that the underlying's value will continue fluctuating after the exercise date needs to be taken into account.

For a date after the exercise date $$t > T$$ and before the underlying's maturity:

$$
\begin{aligned}
MV_\text{option}(t) &= \mathbb{E} \left[ \underbrace{1_{\{MV_\text{underlying}(T) > 0\}}}_\text{exercise outcome} \times \underbrace{\sum_{u > t} D(t,u) CF(u)}_\text{underlying disc. cash-flows} | \mathcal{F}_t \right] \\
&= \underbrace{1_{\{MV_\text{underlying}(T) > 0\}}}_\text{exercise outcome} \times \underbrace{ \mathbb{E} \left[ \sum_{u > t} D(t,u) CF(u) | \mathcal{F}_t \right]}_\text{underlying's value} \\
&= 1_{\{MV_\text{underlying}(T) > 0\}} \times MV_\text{underlying}(t)
\end{aligned}
$$

The exercise outcome is computed at the exercise date $$T$$, and used to get the option's price for dates beyond $$T$$.

Notable examples of options with physical delivery are:

## Late-delivery FX options

A first example is that of a late-delivery foreign-exchange option. For this type of options, the underlying is an FX forward contract. 

For a call option for example, and denoting $$D > T$$ the delivery date, its value after the exercise date $$T$$ is given by:

$$
\begin{aligned}
MV_\text{late-delivery FX call}(t) &= 1_{\{MV_\text{Forward}(T) > 0\}} \times MV_\text{Forward}(t), \quad t > T \\
&= 1_{\{FXFwd(T, D) > K\}} \times P(t, D) \times \left( \mathbb{E}^D \left[ FX(D) | \mathcal{F}_t \right] - K \right) \\
&= 1_{\{FXFwd(T, D) > K\}} \times P(t, D) \times (FXFwd(t, D) - K)
\end{aligned}
$$

In this expression:
- $$FXFwd(t, D)$$ denotes the forward FX rate at $$t$$ for delivery date $$D$$. It can be computed using the FX rate and the domestic and foreign zero-coupon bonds as follows:

$$
FXFwd(t, D) = FX(t) \frac{P_\text{foreign}(t, D)}{P_\text{domestic}(t, D)}
$$

- we used the fact that it is a martingale under the domestic $$D$$-forward measure, associated with the domestic zero-coupon bond with maturity $$D$$ as its numéraire:

$$
\begin{aligned}
\mathbb{E}^D \left[ FX(D) | \mathcal{F}_t \right] &= \mathbb{E}^D \left[ FXFwd(D, D) | \mathcal{F}_t \right] \\
&= FXFwd(t, D)
\end{aligned}
$$

## Swaptions with physical delivery

A second example is the swaption with physical delivery, whose underlying is an interest rates swap.

The value of the payer swaption for example after the exercise date $$T$$ is given by:

$$
MV_\text{payer swaption}(t) = 1_{\{MV_\text{payer swap}(T) > 0\}} \times MV_\text{payer swap}(t), \quad  t > T
$$

The following figure shows two paths of the values of the physical-delivery swaption that was shown at the end of the [previous post](https://alefanalytics.org/blog/2022/01/17/xva-2-typical_exposure_profiles.html), with its underlying (forward start) swap: one where exercise took place and another where it did not.
The simulation was run as of Dec 15th 2021 with exercise date of the swaption on the Dec 15th 2022. This latter is indicated in the plots by the vertical grey line.

{% include plots/late_delivery/swaption_paths.html %}

As expected, on the path where the underlying swap's value is positive at exercise date, the swaption is exercised and its value becomes identical to the swap until maturity, while on the others, it drops to zero.

As it is the case for path 10 displayed above, or some of the paths where no exercise took place, the swap's value will go back into positive territory after the exercise date and will contribute to the swap's EE and PFE but not to the swaption's.

This explains why the EE and PFE of the underlying swap are higher than the swaption's after exercise date as shown below.

{% include plots/late_delivery/swaption_ee_pfe.html %}

# Late-cash options

Another type of options leading to the same considerations are late-cash FX option. Here, the underlying is the FX rate minus the strike, just like a vanilla option. However, the payment is made at a later date $$L > T$$.

This leads to the following expression for its value after the exercise date $$T$$:

$$
\begin{aligned}
MV_\text{late-cash FX call}(t) &= 1_{\{FX(T) > K \}} \times P(t, L) \mathbb{E}^L \left[ FX(T) - K | \mathcal{F}_t \right], \quad t > T \\
&= 1_{\{FX(T) > K \}} \times P(t, L) \left( FX(T) - K \right)
\end{aligned}
$$

Here, unlike the late-delivery FX option, the payoff is fully known at the exercise date $$T$$, and the option turns into a simple cash flow paid at the late payment date on the paths where exercise took place.