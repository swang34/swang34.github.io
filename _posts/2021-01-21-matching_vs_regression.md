---
layout: post
title:  "Matching vs Regression"
categories: blog
date: 2021-01-21
---

[*Reference：Mostly Harmless Econometrics: An Empiricist's Companion,p69-p80*][jekyll-docs1] 

## Motivation

I've obvserved in certain scenarios, matching and regression produced very different results for treatment effect estimates. Why does this happen? What's the difference and which one should I choose? With these questions in mind, this post is a conceptual introduction of Matching vs Regression estimand for causal inference. 

## Contents:
1. *What's matching technique?* \
  *1.1. Example \
  1.2. Conditional Independence Assumption*
2. *Comparison of Matching vs Regression Estimands*
3. *Why are matching and regression estimates sometimes different?*


## TL;DR 
Matching and regression are both essentially control strategies and both rely on the Conditional Independence Assumption to have a causal interpretation. Yet, **matching uses frequency-weighted average and regression uses variance-weighted average to produce the final estimate.** This causes the difference, unless the treatment effect is the same across covariates. The choice of method is a more complicated topic and depends on the underlying assumption on the weighting scheme. 

## What's matching
Matching is a common strategy used for causal inference analysis. The idea is simple. It calculates convariate-specific treatment-control comparisons, weighted together to produce the average teratment effect. 

For example, we'd like to understand the effect of a college degree on wage income. The covariates include age, gender, parents' income, etc. Controling for these factors, the treatment-control difference is the covariate-specific effect. The average treatment effect for treated could then be obtained by the weighted sum of the covariate-specifc effects. 

### Example 
For simplicity, let's assume we have a binary treatment $$D$$, a single categorical covariate $$X$$, and the variable of interest $$Y$$. The *true* relationship is $$Y = 2X + 2D$$ when $$X = 0$$, and $$Y = 2X + D$$ when $$X = 1$$. Below is an example table with 5 observations.

| X | D | Y |
|-------|--------|---------|
| 0 | 1 | 2 |
| 0 | 0 | 0 |
| 1 | 1 | 3 |
| 1 | 1 | 3 |
| 1 | 0 | 2 |

**The primary paramter we are interested in is the treatment effect on the treated, i.e $$E[Y_{1i} - Y_{0i}|D_i=1]$$.** Here, 
- $$Y_{1i}$$: Actual $$Y$$ for those who were treated
- $$Y_{0i}$$: The counterfactual $$Y$$ for those who were treated, had they not been treated
- $$D_i$$: Dummy variable for treatment

For $$X = 0$$, the mean treatment-control difference is 2. For $$X=1$$, the  mean treatment-control difference is 1. The matching esitmand can be calculated as $$2 * P(X=0 \vert D=1) + 1 * P(X=1 \vert D=1)=1.33$$. 

**Note that we will have a biased result if we simply compare the $$Y$$ between those with $$D=1$$ and $$D=0$$** (i.e. 3-2 =1) This is because 

$$
E[Y_{1i}|D_i = 1] - E[Y_{0i}|D_i = 0] \\
=  E[Y_{1i}-Y_{0i}|D_i = 1] + E[Y_{0i}|D_i=1] - E[Y_{0i}|D_i = 0] 
$$

The first term $$E[Y_{1i}-Y_{0i} \vert D_i = 1]$$ is the true treatment effect. The rest is the selection bias. Where does it come from? In the example of college degree, one bias could be that those who received a college degree are more likely to come from a wealthy family that have connections for a higher-paid job. So even though they didn't attend college in the counterfactual world, their wage could still be higher than those who couldn't afford college education. 

The next part shows how to derive the unbiased estimand used to calculate the 1.33 above.

### Conditional Independence Assumption (CIA)
First, we need the CIA assumption. It states that given the covariates $$X$$, the assignment of treatment is indepedent of true value of interest $$Y$$. 

$$
\{Y_{1i},Y_{0i}\} \perp\!\!\!\perp C_i|X_i
$$

Given CIA, the selection bias is zero given X. The average treatment effect on the treated could be computed by iterating expectations over X.  

$$
E[Y_{1i}|D_i = 1] - E[Y_{0i}|D_i = 1] \\
= E\{E[Y_{1i}-Y_{0i}|X_i,D_i=1]|D_i=1\} \\
= E\{E[Y_{1i}|X_i,D_i=1]|D_i=1\} - E\{E[Y_{0i}|X_i,D_i=1]|D_i=1\} 
$$

By CIA, $$Y_{0i}$$ is the same regardless of $$D_{0i}$$ given X, hence 

$$
E[Y_{0i}|X_i,D_i=1] = E[Y_{0i}|X_i,D_i=0]
$$ 

So 

$$
E[Y_{1i}|D_i = 1] - E[Y_{0i}|D_i = 1] \\
= E\{E[Y_{1i}|X_i,D_i=1]|D_i=1\} - E\{E[Y_{0i}|X_i,D_i=0]|D_i=1\} \\
= E[\delta_X|D_i=1] \\
= \sum_{X} \delta_XP(X_i=x|D_i=1) 
$$ 

where 

$$
\delta_X = E[Y_{1i}|X_i,D_i=1] - E[Y_{0i}|X_i,D_i=0]
$$

is the treat-control difference for each covariate $$X_i$$. 

*$$E[Y_{1i}-Y_{0i} \vert D_i = 1] $$ tells us what's the average treatment effect for an average college graduate, while $$E[Y_{1i} - Y_{0i}]$$ tells us what's the average treatment effect for an average exposed user.* 

## Comparison of Matching vs Regression
If you fit the example data into a Linear Regression, you will notice that the coeffienct on $$D$$ is 1.43, slightly larger than the 1.33 computed above. **In fact, regression estimand is in principle some sort of matching estimator, but using a different set of weights to combine covariate-specific effects.** 

While matching uses the distribution of covariates among the treated as weights, regression uses a variance-weighted average of these effects. 

Let $$\beta_D$$ denote the coefficient of treatment $$D$$ in the regression $$Y_i= \sum\limits_{x}d_{ix}\alpha_x+\beta_D D_i+e_i$$ where $$d_{ix}=1$$ is a dummy variable when $$X=x$$ and $$D_i=1$$ is the dummy varible for treatment.


The coefficient $$\beta_D$$ is equal to 

$$ 
\beta_D = \frac{E[\delta_D^2(X_i)\delta_X]}{E[\delta_D^2(X_i)]},
$$ 

where 

$$ 
\delta_D^2(X_i) \equiv E[(D_i-E[D_i \vert X_i])^2 \vert X_i]
$$

is the conditional variance of $$D_i$$ given $$X_i$$. (For the detailed proof, see Chapter 3, p.74-75 of Mostly Harmless Econometrics) 

This shows that the regression model produces a treatment variance weighted average of $$\delta_X$$. Since we assumed a dummy treatment, $$\delta_D^2(X_i)$$ is equal to $$P(D_i=1 \vert X_i)(1-P(D_i=1 \vert X_i))$$.  

And the coefficient could be rewritten as 

$$ 
\beta_D =  \frac{\sum\limits_{X}  \delta_X[P(D_i=1 \vert X_i=x)(1-P(D_i=1 \vert X_i=x))]P(X_i=x)}{\sum\limits_{X} [P(D_i=1 \vert X_i=x)(1-P(D_i=1 \vert X_i=x))]P(X_i=x)}
$$ 

Each treatment effect effect for $$X$$ is weighted by $$[P(D_i=1 \vert X_i=x)(1-P(D_i=1 \vert X_i=x))]P(X_i=x)$$. In contrast, the matching estimand can be written as 

$$
E[Y_{1i}-Y_{0i} \vert D_i=1] \
 = \sum_{X} \delta_XP(X_i=x|D_i=1) \
 =  \frac{\sum\limits_{X} \delta_X P(D_i=1 \vert X_i=x)P(X_i=x)}{\sum\limits_{X}P(D_i=1 \vert X_i=x)P(X_i=x)}
$$ 


The second equation is obtained from Bayes' theorem. Therefore the regression and matching esitmates are different, unless the treatment effect is the same across the covarites and the weighting doesn't matter. 

## Why is the Matching estimand smaller than Regression estimand in this example? 

From above, we see that the matching esitmand puts most weights on covariates that are mostly likely to be treated; regression puts the most weight on covariates where the conditional variance for treatment is the largest. When treatment is binary, the variance is maximized when $$P(D_i=1 \vert X_i=1)=0.5$$. This means that most weights goes to covariates that receive the most balanced treatments. 

In this example, the probability of treatment is 2/3 for $$X=1$$ and 1/2 for $$X=0$$. This causes matching estimator to assign more weights to $$\delta_{X=1}$$ which is 1, and less weights to  $$\delta_{X=0}$$ which is 2. Hence the result calculated from matching estimand is smaller than regression. 

**So, which one should I choose for causal inference analysis?** I don't have a good answer for it and I believe there's no one-size-fits-all answer to this question. To some extent, it depends on your assumption on how to assign weights to different covariates when there's hetergeneous treatment effect. Do you want your estimate to reflect more on the frequency of treatment in observational data? Or do you want it to be more "causal" by leaning towards the estimate coming from the factors with a more balanced assignment? 

It's also important to call out that the idea of matching itself is a very broad concept and matching can be combined with regression to do better. For exmaple, we can perform regression on matched data. It's not an either-A-or-B choice. Jeff Smiths has a [*blog on more matching vs regression on methodology*][jekyll-docs2] for further reading. 

[jekyll-docs1]: https://www.amazon.com/Mostly-Harmless-Econometrics-Empiricists-Companion/dp/0691120358/ref=sr_1_2?crid=2VU0E4I4UOS4B&dchild=1&keywords=mostly+harmless+econometrics&qid=1610933432&sprefix=mostly+harmles+%2Caps%2C207&sr=8-2

[jekyll-docs2]: http://econjeff.blogspot.com/2010/10/on-matching.html

