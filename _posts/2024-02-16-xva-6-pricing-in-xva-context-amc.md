---
layout: post
title: "XVA series - 6. Pricing in the CCR/XVA context: American Monte Carlo"
tags: [XVA, CVA, Counterparty credit risk]
comments: false
---

As seen in previous articles of this series, the computation of XVA and counterparty credit risk metrics such as the expected exposure (EE) and potential future exposure (PFE) profiles requires a grid of market values at future dates and under various Monte Carlo scenarios.

This article presents American Monte Carlo, that is used to compute these market values when a closed-form evaluation formula is not available.

* TOC
{:toc}

# American Monte Carlo Method

## General principle

American and Bermuda style options valuation is a challenging problem in finance. Unlike European options, they have many exercise dates. 

The pricing is done assuming that the option holder will exercise at the optimal exercise date. Denoting $$t_1<t_2<\dots<t_n$$ the exercise dates, the option value has the following expression:

$$
V(0) = \max_{1 \leq k \leq n} \mathbb{E}\left[ e^{-\int_0^{t_k} r(u)du} \text{Payoff}(t_k) \right]
$$

To compute it, at each exercise date $$t_i$$ starting from the last one, the exercise value given by the payoff of the option $$\text{Payoff}(t_i)$$, is compared with the continuation value $$C(t_i)$$, given by the price of a new option with the remaining future exercise dates: $$t_{i+1}, \dots, t_n$$.

Backward numerical methods such as trees or partial differential equations (PDE) resolution are very well suited to this kind of problem, as conditional probabilities are available at each node thus enabling the computation of the continuation value easily. However, they suffer from two shortcomings. The first one is that they do not work for path dependent options. The second is that they suffer from the curse of dimensionality[^1] which makes them impractical as soon as multiple underlyings or multiple factors are involved.

While Monte Carlo simulation may seem like a good alternative as it does not suffer from these limitations, it is not adapted to the computation of conditional expectations as each Monte Carlo path lives its own life and conditional probabilities are unavailable. This is where American Monte Carlo _a.k.a._ least-square Monte Carlo comes into play. 

This technique was first introduced by (Longstaff and Schwartz 2001)[^2]. It relies on approximating the conditional expectations using regression. The idea is to extract the conditional expectation at each date from the cross-sectional information in all simulated paths. Starting from the last exercise date and moving backwards, the continuation value is approximated by regression on one or many observable depending on the state variables.

## Example: The Bermuda swaption

To illustrate, let us consider a payer Bermuda swaption with maturity $$t_n$$ and exercise dates $$t_1 < t_2 < \dots < t_n$$, and use the following notation:

- $$C(t)$$ is the continuation value. It is the value of a new swaption with the remaining exercise dates beyond $$t$$;
- $$E(t)$$ is the exercise value. It is the value of the underlying swap, computed using a closed-form formula[^3];
- $$V(t)$$ is the value of the swaption;
- $$X(t)$$ is the regressor, for simplicity we choose the zero-coupon bond with the same maturity as the swaption.

The procedure goes as follows:

**1. Forward phase**

First, the diffusion is done to get the interest rate model state variable values at each date and path. 

These state variables are then used to compute the exercise value and the regressor at each simulation date and exercise date.

**2. Backward phase**

Second, a backward loop is done on all exercise dates starting from the last $$t_n$$ and until the first $$t_1$$.

For $$t_n$$, the continuation value is zero, so the value of the swaption is easy to compute:

$$
V(t_n) = \max(E(t_n), 0)
$$

At $$t_k$$, assuming $$V(t_{k+1})$$ has been computed in the previous step, $$C(t_k)$$ is approximated by a function $$f_{t_k}$$ of the regressor value at that date $$X(t_k)$$. This is done by discounting $$V(t_{k+1})$$ to $$t_k$$ and regressing it on $$X(t_k)$$:

$$
e^{-\int_{t_k}^{t_{k+1}} r(u)du} V(t_{k+1}) \approx f_{t_k}(X(t_k)) \\
\implies C(t_k) = \mathbb{E} \left[ e^{-\int_{t_k}^{t_{k+1}} r(u)du} V(t_{k+1}) | \mathcal{F}_{t_k} \right] \approx f_{t_k}(X(t_k))
$$

The following graph shows an example of such a regression of discounted value on the regressor at one of the exercise dates.

{% include plots/amc_xva/regression.html %}

This approximated value is compared with the exercise value at $$t_k$$ to get the swaption value at that date:

$$
V(t_k) = \max(C(t_k), E(t_k)) \approx \max(f_{t_k}(X(t_k)), E(t_k))
$$

{% include plots/amc_xva/exercise_frontier.html %}

This operation is repeated until reaching the first exercise date $$t_1$$. Discounting $$V(t_1)$$ back to today and taking the average on all Monte Carlo paths gives the price today $$V(0)$$.

<!--To illustrate, let us take the example of a put option on basket $$B$$ and denote $$C(t_i)$$ and $$E(t_i)$$ the continuation and exercise values at date $$t_i$$. 

At $$t_n$$, $$C(t_n) = 0$$ as it is the last exercise date, while $$E(t_n) = (K - B(t_n))^+$$. So, the value at that date is simply: 

$$
V(t_n) = (K - B(t_n))^+
$$

At $$t_i$$, assuming $$V(t_{i+1})$$ has been computed in the previous step, $$C(t_i)$$ is approximated by discounting $$V(t_{i+1})$$ to $$t_i$$ and regressing it on $$B(t_i)$$ for example:

$$
e^{-\int_{t_i}^{t_{i+1}} r(u)du} V(t_{i+1}) = f(B(t_i)) + \varepsilon \\
\implies C(t_i) = \mathbb{E} \left[ e^{-\int_{t_i}^{t_{i+1}} r(u)du} V(t_{i+1}) | \mathcal{F}_{t_i} \right] = f(B(t_i))
$$

This value is compared with the exercise value at $$t_i$$ to get:

$$
V(t_i) = \max(C(t_i), E(t_i))
$$

This operation is repeated until reaching the first exercise date $$t_1$$. Discounting $$V(t_1)$$ back to today and taking the average on all Monte Carlo paths gives the price today $$V(0)$$.

At the end of this procedure, we get two things:

- the optimal exercise date for each Monte Carlo path;
- the price of the option today $$V(0)$$.-->

# American Monte Carlo in the XVA context

## How is it different?

As seen in previous articles, CCR and XVA computations rely on a hybrid multi-asset Monte Carlo simulation, and require a market value grid. The market value at each simulation date and path is a conditional expectation on the state of the world at that node, given by the diffused model state variables.

Closed-form formulae are important in this case as they enable a swift and efficient computation of the market value grid. When they are not available, for example for exotic trades or when using sophisticated models, using a nested Monte Carlo or PDE numerical resolution (*i.e.* doing a MC or PDE resolution at each date and path) would be too costly in terms of performance. American Monte Carlo is used instead.

In the XVA or CCR context, the technique is the same as explained in the previous section, with the exception that not only exercise dates need to be considered (to determine the optimal exercise strategy), but all simulation dates (to compute the market values).

The choice of the regressors and the regression function itself is crucial for XVA and CCR computation. Even more so for the PFE, for which the tails of the distribution need to be modelled precisely.

For a discussion of American Monte Carlo in XVA context, the reader can refer for example to (Cesari et al 2010)[^4].

## Back to our example: The Bermuda swaption

Let us go back to the Bermuda swaption introduced previously, with the same notation, and assume now that we want to compute the market value grid on a set of Monte Carlo simulation dates $$\{s_i, 1 \leq i \leq m\}$$ and paths.

**1. Forward phase**
The forward phase would be unchanged, with the exception that the exercise value and the regressor are not only needed at the exercise dates, but also at the simulation dates.

**2. Backward phase**
Similarly, for the backward phase, approximation of the continuation value is needed not only at each exercise date, but also at each simulation date:

$$
C(s_i) \approx f_{s_i}(X(s_i)), 1 \leq i \leq m
$$

Furthermore, when determining the value by comparing the exercise and continuation values at each exercise date, the exercise outcome $$e_k$$ is now stored:

$$
e_k = \mathbb{1}_{C(t_k) < E(t_k)}
$$

At the end of this phase, in addition to the price of the swaption today $$V(0)$$, we have:

- the exercise outcome at each date $$t_k$$, assuming no exercise before.
- a function $$f_{s_i}$$ approximating the continuation value for each simulation date $$s_i$$.

But we need an extra phase to get our market value grid.

**3. MV grid computation phase**

The first thing here is to determine the optimal exercise date $$\tau$$ on each path. This is simply the smallest exercise date for each path:

$$
\tau = \min \{e_k | e_k = 1, 1 \leq k \leq n \}
$$

Now, the market value of the Bermuda swaption can be computed on each simulation date $$s_i$$ as follows:

$$
V(s_i) = \mathbb{1}_{\{s_i \geq \tau\}} \times E(s_i) + \mathbb{1}_{\{s_i < \tau\}} \times C(s_i)
$$

- On the paths where the exercise already took place before $$s_i$$, the swaption has become a swap with value $$E(s_i)$$;
- while on the others, it is still a Bermuda swaption with value $$f_{s_i}(X(s_i))$$ previously computed.

For the cash swaption, instead of keeping the swap value after exercise date, the value drops to zero as the full MV is received upon exercise.

Taking the positive part of the market value and taking the average (resp. the quantile) gives us the EE (resp. PFE) profile of the cash and physical Bermuda swaptions:

{% include plots/amc_xva/ee_pfe_profiles.html %}


Similarly to European swaption profiles introduced in [the second article](https://alefanalytics.org/blog/2022/01/17/xva-2-typical_exposure_profiles.html) of this series, we can see that:

- The profiles are identical for the cash and physical Bermuda swaptions before the first exercise date;
- The profiles of the cash Bermuda swaption drop to zero after the last exercise date;
- In between, the profiles of the cash Bermuda swaption drop quicker than the physical's towards zero as its value falls to zero on exercised paths.
 
<!--### Forward phase

First, the diffusion is done to get the interest rate model state variable values at each date and path. 

These state variables are then used to compute the exercise value and the regressor at each simulation date and exercise date.


### Backward phase

Second, the backward phase is done starting from the biggest date and moving backwards on the set of dates including:

- the exercise dates $$(t_k)_{1 \leq k \leq n}$$;
- all simulation dates $$(s_i)_{1 \leq i \leq m}$$ (XVA specific).

At each simulation date $$s_i$$ the regression function giving the continuation value as a function of the swap rate $$SR$$ is fitted and stored to be used later.

$$
C(s_i) \approx f_{s_i}(X(s_i))
$$

At each exercise date $$t_k$$, the same operation is done:

$$
C(t_k) \approx f_{t_k}(X(t_k))
$$

Then it is used to determine the exercise outcome. Denoting $$\tau$$ the exercise date, that is:

$$
\begin{aligned}
& V(t_k) = \max (C(t_k), E(t_k)) \\
& t_k \in \text{Exercise dates} \iff C(t_k) < E(t_k) \\
\end{aligned}
$$

We move to the previous date (backwards), and discount the value of the swaption and recompute the value of the swap:
$$

$$
-->

# Conclusion

As we have seen in this article, American Monte Carlo enables to compute conditional expectations in a Monte Carlo simulation. While it is not the best method for pricing American and Bermudan style options on a single underlying, it becomes very useful for multi-underlyings or in the presence of path-dependency. It is also the method of choice for XVA and CCR computations, not only for trades with American or Bermudan exercise features, but more generally for all trades for which a closed-form evaluation is not available. 

AMC consists in the following 3 phases:

1. Forward phase:
	- Diffusion and computation of any market observables, including the regressors, from the model state variables. 
2. Backward phase:
	- Discount of the swaption value while moving backwards.
	- Regression functions fit at: 
	    + each exercise date, to determine the continuation value.
		+ each simulation date, to determine the continuation value.
	- Determination of the exercise outcome at each exercise date, and computation of the swaption value.
3. MV grid computation phase:
	- Optimal exercise date determination for each path.
	- At each simulation date $$s_i$$, evaluation of the market value using the functions fitted previously and the exercise outcomes:
	    + exercise value for the paths where the exercise took place before $$s_i$$.
		+ continuation value for the others.

Given the important role it plays in this technique, the choice of the regression method and regression basis is crucial. Especially for the PFE that is very sensitive by the behavior of the regression at the tails.

# Footnotes and References


[^1]:
	The size and computational cost increase exponentially with the number of factors. Assuming a grid of size $$N$$ is neeeded to resolve the PDE for 1 factor, the size becomes $$N^d$$ for d factors. 
	
[^2]:
	{% include citation.html key="longstaff" %}

[^3]:
	The exercise value could also be computed by regression, similarly to the continuation value.

[^4]:
	{% include citation.html key="cesari" %}	
	