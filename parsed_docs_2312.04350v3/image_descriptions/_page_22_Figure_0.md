# _page_22_Figure_0.jpg

**Source:** `assets/_page_22_Figure_0.jpg`

**Generated:** 2026-05-16 13:58:17

---

The image is a diagram illustrating the three rungs of causal inference in a causal graph. It consists of three main sections:

1. **Rung 1. Association**: This section explains the correlation between variables X and Y, represented as P(Y|X). It highlights two paths: a direct causation path and a backdoor path through a confounder Z.

2. **Rung 2. Intervention**: This section describes how direct intervention on X cuts off all its parents, leading to the average over all non-descendants of X to get P(Y|do(X)).

3. **Rung 3. Counterfactuals**: This section discusses how to force X to be the counterfactual value x, isolating the effect of X by looking at the counterfactual P(Y|X=x). It emphasizes the need to consider all non-descendants of X as if X were still the original value x.

The diagram uses arrows to represent causal relationships and includes labels such as "do(X)" and "N_X" to denote intervention and non-descendants, respectively.