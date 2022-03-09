---
layout: post
title: "XVA series - 3. On the consistency between diffusion and pricing models"
tags: [XVA, CVA, Counterparty credit risk]
comments: true
---

In the previous articles, we have seen that the diffusion of risk factors and pricing of the trades are two key steps in XVA computations.
In this one, we will see the arguments for or against each of these two possible approaches: the *inconsistent approach*, where the dynamics are different between diffusion and pricing, and the *consistent approach*, where the same dynamics are used for both diffusion and pricing.


* TOC
{:toc}

# Notation and abbreviations

Throughout this article, the following notation and abbreviations will be used:

- $$\beta(t)$$: the money market account numéraire value at $$t$$, defined by: $$\beta(t) = e^{\int_0^t r(u) du}$$ where $$r$$ is the short rate.
- $$P(t, T)$$: the value at $$t$$ of the zero-coupon bond with maturity $$T$$.
- $$FXFwd(t, T)$$: The forward foreign exchange rate at $$t$$ for delivery at $$T$$
- $$LGD$$ : the loss given default.
- $$NDP(t)$$ : the non-default probability between today and date $$t$$.
- $$X^+$$: the positive part of quantity $$X$$, defined as: $$ X^+= \max(X, 0)$$.
- $$V(t)$$ : the market value at date $$t$$.
- $$EE(t)$$ : the expected exposure at date $$t$$.
- $$\mathbb{E}$$: the expectation under the risk-neutral measure, associated with numéraire $$\beta(t)$$.

# Inconsistent approach

The base price of each trade, computed by each trading desk and not including the XVA adjustments (let’s call it the front office price or *FO price*) uses a model and calibration basket that are specific to the trade.

Taking foreign exchange options as an example, vanilla options are valued using Black Scholes model whereas barrier options are typically valued using stochastic local vol models.

Within the same asset class, and even within the same trade type, the calibration baskets can be different. As an example, for each Bermuda swaption, the calibration is done on co-terminal swaptions with the same
maturity, and with expiries corresponding to its exercise dates.

If the consistent approach is used for XVA, then a single model (the one used for diffusion) is used to value all trades. Moreover, a single, trade-independent calibration is used for this model, leading to trade prices that are different from the FO.

It is important to note however that these differences are only observed for optional trades. Indeed, as today’s prices of linear trades are not impacted by the pricing model choice, they are identical between XVA and FO.

So, in the presence of optional trades, today’s exposure used for the XVA computation will not match the actual one.

In order to reduce this mismatch, XVA pricing models must be trade-specific and as close as possible to the FO ones. In particular, they must be independent from the diffusion models.
This approach would lead to close exposures and consistent sensitivities between CVA and FO, thus giving more confidence to traders and sales in the accuracy of the XVA values.

For example, if the FO pricing model for a given trade is sensitive to the volatility smile, while the CVA diffusion one is calibrated only on at-the-money volatility, then the trade value will have a mismatch and it will not exhibit any volatility smile sensitivity unless the pricing model used for CVA is similar to the FO one.

Models used on FO side can be very complex and require computation-heavy numerical methods to get the price. 

These models cannot be used for XVA computation as it requires multiple pricings at various future dates and for thousands of Monte Carlo scenarios.
Taking the example of an exotic trade whose pricing takes 1 second, with 100 time steps and 10,000 Monte Carlo scenarios which are typical for XVA calculations, more than 4 hours of computation time would be required on a 64 core CPU to get the XVA figures, and an order of magnitude higher for their sensitivities.
This is unacceptable as the pricing is needed in near-real time for incremental XVA computation, and that the end of day XVA computation can include tens of thousands of trades, some of which will be exotics.

As a result, simpler models are often favored for XVA. This means that the choice of inconsistency between diffusion and pricing does not always guarantee having the same exposure on XVA and FO sides.

# Consistent approach

The XVA terms are equivalent to complex hybrid derivatives. The CVA for example can be seen as a weighted sum of options on the whole counterparty trades, with strike zero and with different expiry dates (with netting and no collateral, with collateral it's even more complex) :

$$
\begin{aligned}
CVA &= \sum_{i=1}^n w_i \mathbb{E}\left[ \frac{V(t_i)^+}{\beta(t_i)} \right] \\
w_i &= LGD \left( NDP(0, t_{i-1}) - NDP(0, t_i)\right)
\end{aligned}
$$

From a theoretical perspective, the use of the same models for trades valuation and risk factors diffusion is required to ensure the absence of arbitrage and the mathematical soundness of the Monte Carlo simulation.

This also implies a single model calibration, which makes the XVA computation easier to understand.
This is especially the case for hedging purposes and for the daily P&L explain, as all sensitivities and variations come from a single calibration.

To illustrate the mathematical validity of the consistent approach, two examples are presented:

## Example 1: Martingale property of discounted prices

The risk-neutral probability measure is very useful for derivatives valuation (and we have seen that the XVAs are complex derivative) as it permits to compute the price as an expectation of the discounted payoff.
This is because the discounted prices are martingales under this measure.

In a Monte Carlo simulation such as the one used for XVA, this martingale property holds only when the dynamics are consistent between diffusion and pricing. 
Otherwise, it could break down as shown next.

The figure bellow shows the expectation of the discounted price of a zero-coupon bond with 3Y maturity at various points in time. 
In the consistent case, the plot is a straight line, showing that the expectation is constant and that the martingale property holds.

In the inconsistent case on the other hand, the line decreases before increasing back to its initial value.

<!-- {% include aligner.html images="posts/martingality_of_zc_price.PNG" column=1 caption="Expectation of the discounted value of a 3Y ZC bond" %} -->

{% include plots/consistency/martingality_of_zc_price.html %}

The two curves start at the same point because the pricing model parameters have no influence on the zero-coupon bond price at $$t = 0$$. It is only given by the input discount curve.

They also coincide at the $$t = 3Y$$ point because, as the zero-coupon bond reaches its maturity, its value is known and equal to $$100\%$$. The pricing model is not involved anymore.

## Example 2: Consistency of future expected exposures with today prices

Another example to assess the validity of the Monte Carlo simulation is to check the consistency of future expected exposures of linear trades, with the price today of their optional equivalents.

To illustrate this point, let us consider an FX forward trade with delivery date T and strike K.
At a given pricing date t, its price is given by: 

$$
V_\text{Forward}(t) = P(t, T) \left(FXFwd(t, T) - K \right) 
$$

This is also the payoff at date $$t$$ of a late delivery FX option, with expiry at $$t$$ and delivery at $$T$$:

$$
V_\text{Forward}(t) = Payoff_\text{Option}(t)
$$

Dividing by the money market account numéraire on both sides and taking the risk-neutral expectation, we get the following result:

$$
EE_\text{Forward}(t) = V_\text{Option}(0)
$$

The expected exposure of the FX forward at each pricing date $$t$$ should be equal to the price today of an FX option expiring at $$t$$ and having the same underlying, strike K and delivery date $$T$$:

This relationship will only hold if the choice of consistency is made. The following figures illustrate this for a 3Y EURUSD forward.

First, in the consistent case, we can see that the relationship holds and that the EE profile of the FX forward matches the value of the corresponding FX option:

<!-- {% include aligner.html images="posts/consistent_ee_vs_mv.png" column=1 caption="FX forward EE vs. FX option price - Consistent case" %} -->
{% include plots/consistency/consistent_ee_vs_mv.html %}

In the inconsistent case, the options prices today are higher than the forward’s expected exposure because the volatility used for pricing is higher than the one used for diffusion.

<!-- {% include aligner.html images="posts/inconsistent_ee_vs_mv.PNG" column=1 caption = "FX forward EE vs. FX option price - Inconsistent case" %} -->
{% include plots/consistency/inconsistent_ee_vs_mv.html %}

# Pros and cons summary

The table bellow summarizes the pros and cons of each approach:

|---------------|---------------------------------------------|--------------------------------------------------|
|Approach       | Pros                                        | Cons                                             |
|---------------|---------------------------------------------|--------------------------------------------------|
|*Consistent*   | - Theoretical soundness <br> - Sensitivities and PnL easier to explain <br> - Better performance | - Price mismatches with FO for optional trades <br> - Inconsistent sensitivities with FO |
|---------------|---------------------------------------------|--------------------------------------------------|
|*Inconsistent* | - Closer prices to FO for optional trades <br> - More flexibility <br> - Consistent sensitivities with FO  | - Theoretical soundness <br> - Sensitivities and PnL harder to explain <br> - Poor performance |
|---------------|---------------------------------------------|--------------------------------------------------|

# What about CCR computations (e.g. PFE)?

It is important to note that the considerations presented in the previous sections of this document apply only to the XVA. They are not relevant for counterparty credit risk metrics such as the PFE.

Indeed, the PFE is used for limits control, which requires realistic dynamics for risk factors diffusion during its computation. It is a quantile corresponding to the market value under an extreme scenario, not an expectation. Furthermore, usually it is not discounted back to today.

The diffusion models used for CCR computations are calibrated historically. Also, backtesting of risk factor evolution, trade and portfolio PFE should be performed, where the realized trajectories are compared against the model’s.

On the other hand, the price at each node of the Monte Carlo simulation is a conditional expectation under the risk-neutral measure. As a result, the pricing cannot be done using historically calibrated models.

In conclusion, by construction, counterparty credit risk models are always inconsistent between diffusion and pricing.

# Conclusion

The consistent approach is the only one that is mathematically sound for XVA valuation. To reduce the mismatch between XVA and FO prices resulting from this approach, more sophisticated models should be used with richer calibration instruments.
This comes however at the cost of performance degradation and implementation complexity. 

So, the right balance should be found, and the mismatches between FO and CVA pricing closely monitored, with actions in case they become too high and exceed tolerances that can be set by trade, trade type, counterparty, etc.

# References

- {% include citation.html key="green" %}