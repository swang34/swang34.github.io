---
layout: "post"
title:  "Causal Inference: Chapter4"
categories: "blog"
---

### Effect Modification / Heterogeneous Treatment Effects 
Sometimes, the null hypothesis that there's no causal effect is true for the whole population , but not necessarily for certain sub-populations. 

Notation: A represents treatment, and L represents critical condition, which determines the probability of a patient receiving a heart transplant. 

For a binary outcome, it means Pr[Y(a=1) = 1|V = 1] != Pr[Y(a=0) = 1|V = 1]. To confirm whether there's effect modificatin, we can conduct a stratified analysis in 2 stages. 
1. Construct stratification by V 
2. In side each strata, do standarization by L (or, IP weighting with weights depending on L)

Note: There's an essential difference between L and V. V is an indicator for potential heterogeneous treatment effect (not necessarily causal for the HTE), while L is conditional probability of a patient receiving the treatment. It's possible that L is only correlated with A, but not to Y. 

### Matching as a nother form of adjustment 

This is what matching does: For each patient with A = 0, L = 0, randomly match her with another patient with A = 1, L = 0. We will have a matched pair with L being the matching factor. 

In practice:
* One often chooses the group with fewer individuals.
* Matching doesn't have to be one-to-one. It could be one-to-many. (matching set)
* L is a vector of several varaibles. 

Beucase the matched population is a subset of original study population, the causal effect of the matched population will generally differ from the original, unmatched population. 
