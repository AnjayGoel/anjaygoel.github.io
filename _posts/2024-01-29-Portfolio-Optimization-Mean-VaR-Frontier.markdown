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







But now that I look back at it as a real investor, I find it unsuitable for real life applications, out of the box for a
few reasons.

* Firstly, standard deviation is a relatively unintuitive risk measure. I get no information when someone says
  that the std-dev of their portfolio is blah-blah.
* Secondly, I will be more concerned about my potential losses (the tail risk,
  as we call it in finance) rather than the volatility of my portfolio.

This is where CVaR comes in. The CVaR (at /alpha level) or the expected shortfall (denoted by ES) is simply the loss in
the worst /alpha% of the cases. Its much more intuitive and interpretable, risk measure. So, in this blog I will
construct an efficient using CVaR as a risk measure.

The problem with using CVaR in the optimization is, well we have to calculate CVaR of the portfolio.
The easy way out would be to use the historical data to calculate CVaR. But a better way is to model the returns as a
multivariate distribution and perform monte-carlo simulation.

Obviously, a gaussian distribution is not suited for the task, as we are concerned about the tail risk. Instead, we can
use something called copula's. Which is a nice way to break down the multivariate distribution into corresponding
marginals and a copula distribution modeling the correlation between the assets.

* Insert formula.

### Data

* The data is taken from AMFI (insert link). All mutual funds are direct-growth plan. (Explain what it means)

### Results

* Explain the resultant portfolio and comapare it with one using std-dev.
