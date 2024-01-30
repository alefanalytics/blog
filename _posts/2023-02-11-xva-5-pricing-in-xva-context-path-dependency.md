---
layout: post
title: "XVA series - 5. Pricing in the CCR/XVA context: Path dependent trades"
tags: [XVA, CVA, Counterparty credit risk, Path-dependency]
comments: false
---

In the [previous article](https://alefanalytics.org/blog/2022/12/27/xva-4-pricing-in-xva-context-delivery-options.html) of this series, we illustrated how options with late-cash payment or with late-physical delivery are handled in XVA and CCR computations.

This article present another example of a pricing challenge specific to XVA and CCR context: the path-dependent trades.

* TOC
{:toc}

# Path-dependent trades in XVA and CCR computations

A path dependent option is one whose value depends not only on the value of an underlying asset or market observable at its exercise date, but on the path that it took during all or part of the life of the option.

When computing XVA and CCR metrics for such trades, even if a closed-form pricing formula exists, one should handle the path-dependency at each future pricing date $$t$$ by taking into account what happened along each Monte-Carlo path, from today and until that date $$0 \leq s \leq t$$.

Examples include:

- Default of the reference entity for credit default swaps;
- Average value for Asian options;
- Exercise for American options;
- Knock event for FX barrier options.

# Example: The FX touch option

To illustrate, let us consider an FX touch option with maturity $$T$$, a rebate $$R$$ paid at maturity, and a continuously monitored up barrier $$B > FX(0)$$. Let us denote $$\tau$$ the first time where the barrier is knocked.

This trade will pay the rebate $$R$$ at maturity if the barrier is knocked out between 0 and $$T$$, and nothing otherwise.

After the barrier is knocked out, the value of the trade is easy to compute and is just the discounted rebate, let us denote this value $$MV_\text{rebate}$$.
Suppose that we also have a closed-form formula for the FX touch option before the barrier is knocked, denoted $$MV_\text{touch}$$. 

Simply applying $$MV_\text{touch}$$ to get the value of our FX touch for each Monte Carlo scenario at $$t > 0$$ would be wrong. This is because the barrier might have been already hit before $$t$$ on some of these scenarios. For those ones, the option has indeed transformed into the rebate and $$MV_\text{rebate}$$ must be used instead.

So, the value of the FX touch at $$t > 0$$ is given by:

$$
MV(t) = 1_{\{ \tau > t \}} MV_{touch}(t) + \left(1 - 1_{\{ \tau > t \}}\right) MV_\text{rebate}(t)
$$

where: $$1_{\{ \tau > t \}}$$ is an indicator of no-knock with value 0 for the paths where the barrier was knocked before $$t$$, and 1 for the others.

# No-knock between consecutive simulation dates

In theory, the no-knock indicator $$1_{\{ \tau > t \}}$$ value is known at $$t$$, but in practice, the $$FX$$ spot values are known only on the set of discrete simulation dates before $$t$$.
In the following, we follow Gobet[^1] to approximate these indicators.

Let us denote $$p_i = \mathbb{P} (\tau \in [t_i, t_{i+1}])$$ the probability of knock between consecutive simulation dates: $$t_i < t_{i+1}$$. This probability can be computed by considering that the FX spot follows a Brownian bridge between diffused values $$FX(t_i)$$ and $$FX(t_{i+1})$$.

A first possibility here is to combine these probabilities $$p_i$$ to get the no-knock probability between today and each simulation date $$t_j$$ and use this probability as an approximation of the no-knock indicator in the pricing formula given before:

$$
1_{\{ \tau > t_j \}} \approx \mathbb{P}(\tau > t_j) = \prod_{i=0}^{j-1} \left(1 - p_i \right)
$$

An alternative solution is to simulate the knock event in each interval $$[t_i, t_{i+1}]$$. Since we know the probability $$p_i$$ of this event, this can be achieved by simulating a uniform random number $$U_i \sim \mathcal{U}[0,1]$$ and comparing it to $$p_i$$, leading to the following expression of the no-knock between today and pricing date $$t_j$$:

$$
1_{\{\tau > t_j\}} = \prod_{i=0}^{j-1} \left(1 - 1_{\{U_i < p_i \}} \right)
$$ 

This second method would lead to a bit more variance of the market value, and should lead to a higher PFE.

# Illustration with three Monte Carlo paths

In the following graphs, $$R = 1M$$, $$FX(0) = 1.00$$, $$B = 1.06$$ and $$T = 2Y$$. They show the following for 3 Monte Carlo paths:

- the FX trajectory compared with the barrier.
- the no-knock probability $$\mathbb{P}(\tau > t)$$ and the no-knock indicator $$1_{\{ \tau > t \}}$$.
- the conditional market value $$MV_\text{touch}(t)$$, as well as the unconditional one $$MV(t)$$ using both the no-knock probability: *MV (proba)*, and indicator *MV(indic.)*.

## FX far from the barrier

{% include plots/path_dependent/fx_touch_0.html %}

In this first scenario, the FX spot never gets close to barrier. As a result, the no-knock probability remains equal to 100% over the life of the trade, and its trade value is reduced to $$MV_\text{touch}$$.

Furthermore, the closer we move to maturity, the less probable is the knock event. So, the trade value decreases with time until reaching zero close to maturity.

## Barrier knock-out

{% include plots/path_dependent/fx_touch_46.html %}

In this second scenario, the barrier is knocked out. The no-knock probability gets to zero at the time of the knock, and the trade value becomes $$MV_\text{rebate}$$ for the remainder of its life. $$MV_\text{touch}$$ continues to fluctuate up and down with the FX value, but it becomes irrelevant after the first knock.

Furthermore, although the no-knock probability in Jan 2023 is at 45%, the simulated indicator is equal 1, leading to a higher value when using the probabilities vs. simulating the knock event.

## FX close to barrier without knock-out

{% include plots/path_dependent/fx_touch_85.html %}

In this last scenario, the FX spot gets very close to the barrier. Although the barrier is not knocked out on any simulation date, the no-knock probability still drops, reflecting that it might have been hit in-between, as seen previously.

The closer the no-knock probability gets to zero, the closer the market value gets to $$MV_\text{rebate}$$.

Here, as the no-knock indicator drops to zero directly while the probability drops to 15% and then to 12%, the value using the indicator is higher than the one using the probabilities as an approximation.

# References
[^1]:
    {% include citation.html key="gobet" %}

[^2]:
	See for example: {% include citation.html key="cesari" %}