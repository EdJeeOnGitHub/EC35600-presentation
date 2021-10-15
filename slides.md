---
# try also 'default' to start simple
theme: seriph 
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
defaults:
  layout: 'center'
# Paper Discussion: This is a 10-minute discussion designed to simulate what a discussant would 85 
# present at an economics conference.  Students may choose any of the non-starred papers on the 86 
#  5 
# syllabus.  Students  choosing  other  papers  should  confirm  their  choice  to  us.    The  presentation 87 
# should spend 3-4 minutes summarizing the paper and its empirical strategy.  The next 3-4 minutes 88 
# of the presentation should highlight the paper’s contribution, any criticisms, and avenues for 89 
# further work.  The remaining 2 minutes should be reserved for discussion and questions.
---

# Channeling Fisher: 
randomization tests and the statistical insignificance <br>
 of seemingly significant experimental results


<br>
<br>
Alwyn Young 
<br>
(Presented by Ed)

<!-- <div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div> -->

<!-- <div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div> -->

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---


# Context

- Part of a growing literature on inference and everyone's favourite topic:
standard errors e.g. Betrand, Duflo, Mullainathan (2004); Abadie et al. (2017).


- Loosely related to work by Brodeur et al. (2016)  investigating $p$-values and
  publication bias.


- But also a very old literature - Fisher's _The Design of Experiments_ (1935).


- Similar conclusion to Broderick, Giordano, Meager (2021)'s working paper 
exploring the fragility of OLS results to high-leverage/sample deletion. 

-  Target of a recent [Data Colada "takedown"](http://datacolada.org/99) (of Ariely fame).

- Athey and Imbens (2016) provide an overview of econometrics in randomised 
experiments.
---
layout: two-cols
---

# Contribution


<br>

Young explores  inference used in 53 
experimental AEA papers and proposes the use of randomisation tests over 
regression based inference using standard errors. He finds:
<br>
- 13-22% fewer significant results compared to authors' original methods.
<br>
- After adjusting for multiple treatment effects this falls to 33-49% fewer
significant results.
- Removing one observation in results significant at 1% level leads to 35% 
becoming insignificant.  

::right::

<br>
<br>
<br>
<br>
<br>
<br>

<img src="/p-val-del.png" width = "300">
<!-- ![](/p-val-del.png) -->


---

# What is a Randomisation Test?

It's just like any other test, we need:
  - data
  - a null hypothesis 
  - a test statistic
  - a distribution of the test statistic under the null.

The only novelty of randomisation testing is how we derive the null distribution.


---

# 
<!-- <div v-click-hide> -->
Traditionally, under the null hypothesis we use Slutsky to give us:

$$
T_n = \frac{\sqrt{n}(r' \hat{\beta} - r' \beta_0)}{\sqrt{r' \hat{\Sigma}}r} 
\rightarrow^d N(0, 1)
$$
<!-- </div> -->

<br>
<br>
<br>
<br>
<div v-click>
Instead, randomisation inference creates a null distribution by calculating the test 
statistic under <em>all possible permutations of treatment assignment</em>.
</div>
<!-- <div v-click>Hello</div>
<div v-after>World</div> -->


---
layout: two-cols
---
# an example

```python
n = 1000
epsilon = rnorm(n, 0, 2)
beta = 0.1
D = rbernoulli(n)
y = D*beta + epsilon
experiment_df = tibble(y = y, D = D)
```

---
layout: two-cols
---
# an example

```python
n = 1000
epsilon = rnorm(n, 0, 2)
beta = 0.1
D = rbernoulli(n)
y = D*beta + epsilon
experiment_df = tibble(y = y, D = D)
```

```python
test_stat = function(df){
    treat_df = df[df["D"] == TRUE,]
    control_df = df[df["D"] == FALSE, ]
    t_hat = mean(treat_df[["y"]]) - mean(control_df[["y"]])
    return("test_stat" = t_hat)
}
realised_t_hat = test_stat(experiment_df)
```

---
layout: two-cols
---
# an example

```python
n = 1000
epsilon = rnorm(n, 0, 2)
beta = 0.1
D = rbernoulli(n)
y = D*beta + epsilon
experiment_df = tibble(y = y, D = D)
```

```python
test_stat = function(df){
    treat_df = df[df["D"] == TRUE,]
    control_df = df[df["D"] == FALSE, ]
    t_hat = mean(treat_df[["y"]]) - mean(control_df[["y"]])
    return("test_stat" = t_hat)
}
realised_t_hat = test_stat(experiment_df)
```

```python
permuted_df = permute(experiment_df, 1000, D)

perm_outcomes = map(permuted_df$perm, 
                    ~mutate(
                      experiment_df,
                      D = D[.x$idx]) %>% 
                      test_stat()) %>%
    unlist()
```



---
layout: two-cols
---

# an example

```python
n = 1000
epsilon = rnorm(n, 0, 2)
beta = 0.1
D = rbernoulli(n)
y = D*beta + epsilon
experiment_df = tibble(y = y, D = D)
```

```python
test_stat = function(df){
    treat_df = df[df["D"] == TRUE,]
    control_df = df[df["D"] == FALSE, ]
    t_hat = mean(treat_df[["y"]]) - mean(control_df[["y"]])
    return("test_stat" = t_hat)
}
realised_t_hat = test_stat(experiment_df)
```

```python
permuted_df = permute(experiment_df, 1000, D)

perm_outcomes = map(permuted_df$perm, 
                    ~mutate(
                      experiment_df,
                      D = D[.x$idx]) %>% 
                      test_stat()) %>%
    unlist()
```



::right::
<img src="/perm-table-big.png" width="400" align="right" >
<!-- ![](/perm-table-big.png) -->

---

<img src="/null-dist-2.png" width = "600">

We ask, under the null hypothesis, how many times do we see a more extreme 
test statistic?


---
layout: two-cols
---
## pros

We don't need to use the CLT to approximate a <br>
limiting distribution in asymtopia - the null distribution is _exact_.


Inference provides exact tests "no matter what the sample size, regression design
or characteristics <br>of the disturbance term".


Can generate tests for objects with unknown <br> limiting distributions.


Conceptually, matches the idea of potential outcomes and randomised experiments
nicely.



::right::
## cons



In practice it's difficult to enumerate all possible treatment assignments - 
instead we just randomly draw $N$ permutations in a monte-carlo experiment[^1].


One drawback of randomisation inference is that it tests a _sharp_ null:

$$
H_0 = \beta \quad \forall i
$$

It's not designed to test an $ATE$ but whether treatment has an effect of $\beta$
for all individuals in an experiment.

Young argues this is still a question of importance and simulates how poorly
randomisation inference fares under heterogeneous treatment effects.


<!-- Rate N convergence rather than root N so still probably quicker -->

[^1]: <small>Still a faster rate of convergence than the CLT however.</small>

---

# Results

<!-- ![](/rt-results.png) -->
<img src="/rt-results.png" width = "600">

<!--
Take away from col 1: 21.6% of author's specifications significant.

Of those, only 78% remain significant after randomisation inference.


Look at High leverage col only 65% are still significant.
-->

---

# leverage the culprit?



Young argues a lot of these differences are caused by high leverage observations.


Intuition, with binary treatment:


- 50-50 treat-control -> uniform leverage.
- three treatment arms, 25-25-25-25 (3x treat-control)
  - regress outcome on treatment specific measure (25-75 treat-control split)
- a quarter of observations now account for three quarters of the leverage.  

Experimental design can have a large influence on robustness of results/power etc.




---
layout: two-cols
---

# Data Colada Dispute


A recent blog post suggests randomisation inference isn't really 
necessary. 


<br>
<br>
<br>
<br>



Most of Young's results are driven by using Stata's default
`reg y x, robust` $HC1$ standard errors.

<br>
<br>

Modern standard errors use degrees of freedom and leverage adjustments to 
reduce estimator bias of var-covar matrix, especially in small samples.

::right::

<Tweet id="1448252990687784961" scale="0.8"/>

---

<img src="/data-colada.png">

---

# Conclusion

- Experimental design can have a large impact on power and result robustness.

- Randomisation inference offers a convenient alternative to standard error fights.

- Probably don't use `reg y x, robust` in small samples without specifying `vce`.


<br>
<br>
<br>
Data Colada post probably isn't quite a big a deal as it seems. Same conclusion, different way of getting there?
