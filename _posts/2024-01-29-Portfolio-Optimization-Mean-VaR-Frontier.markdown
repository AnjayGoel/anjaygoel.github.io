---
layout: post
title: "CVaR based portfolio optimization"
date: 2024-01-29 23:00:00 +0530
category: Blog
tags: [ finance ]
desc: "Using CVaR as the risk measure for constructing efficient frontier and portfolio optimization"
description: "Using CVaR as the risk measure for constructing efficient frontier and portfolio optimization"
---

<!--end_excerpt-->

### Intro & Brief Recap

If you have ever been to a Finance-101 class, then perhaps you are familiar with mean-variance portfolio optimization. It is a popular framework to construct an optimal portfolio, i.e., maximize the returns for a given level of risk.

Briefly, a mean variance optimzation task is given by:

$$
\begin{aligned}
\min_{w} w^T \Sigma w \\
\text{s.t.} \quad & w^T \mathbf{1} = 1 \\
& w^T \hat u = \bar r
\end{aligned}
$$

Where $\Sigma$ is the covariance matrix of the returns, thus $w^T \Sigma w$ is nothing but portfolio variance, $\hat
u$ is the returns of the assets, $\bar r$ is the expected returns of the portfolio and $w$ is the weight vector (which is the variable in the optimization).

If we make a scatter plot of this risk - return for variying $w$, we get the efficient frontier, which has the best possible expected return for the given level of risk.

### The problem

Now that I look back at this as a real investor, I find it unsuitable for real life applications, out of the box for a few reasons.

* Firstly, standard deviation is a relatively unintuitive risk measure. I get no information when someone says that the $\sigma$ of their portfolio is blah-blah.
* Secondly, I will be more concerned about my potential losses rather than the volatility of my portfolio.

### The Solution: CVaR

CVaR (which stands for cumulative value at risk) or the expected shortfall (denoted by ES) at $\alpha\%$ level is simply the expected loss in the worst $\alpha\%$ of the cases. Its much more intuitive and interpretable, risk measure.  So, I will construct an efficient using CVaR as a risk measure.



### Estimating CVaR of a portfolio

The challenge with using CVaR in portfolio optimization is, well we have to calculate CVaR of the portfolio. The easy way out would be to use the historical data to calculate CVaR and the returns. But a better way is to model the assets returns as a multivariate distribution and perform monte-carlo simulation.



    Obviously, we cannot simply use a multivariate gaussian distribution, as we are concerned about the tail risk. Instead, we will use something called Copulas. Every multivariate cumulative distribution function can be expressed in terms of its marginals and a copula, i.e.

$Pr[X_1<x_1,X_2<x_2...,X_n<x_n] = C(F_1(x_1),F_2(x_2)..,F_n(x_n))$

Thus allowing us to model the correlations between the variables seperately from the marginal distributions.





### Data

* The data is taken from AMFI (insert link). All mutual funds are direct-growth plan. (Explain what it means)

### Results

* Explain the resultant portfolio and comapare it with one using std-dev.
