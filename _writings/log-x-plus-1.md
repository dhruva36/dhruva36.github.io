---
title: "The Problem with log(X+1)"
date: 2026-04-05
published: false
description: "Why using log(X+1) is a bad specification in econometrics, and why PPMLHDFE offers a better alternative."
toc:
  - id: why-we-log
    title: Why We Log in the First Place
  - id: the-zeros-problem
    title: The Zeros Problem
  - id: what-log-x-plus-1-actually-does
    title: What log(X+1) Actually Does
  - id: the-elasticity-breaks
    title: The Elasticity Breaks
  - id: ppmlhdfe
    title: "PPMLHDFE: The Right Tool"
  - id: a-live-example
    title: "Seeing It in Trade Data"
  - id: beyond-trade
    title: "Beyond Trade: ER Visits with 82% Zeros"
  - id: when-this-matters
    title: When This Matters
footnotes:
  - "Santos Silva and Tenreyro (2006), \"The Log of Gravity,\" <em>The Review of Economics and Statistics</em>, 88(4), 641–658."
  - "The elasticity interpretation of OLS log-linear coefficients assumes the transformation is exact. Adding a constant violates this — the estimated coefficient no longer approximates a percentage change."
  - "PPMLHDFE stands for Poisson Pseudo-Maximum Likelihood with High-Dimensional Fixed Effects. The Stata command was developed by Correia, Guimarães, and Zylkin (2020)."
  - "Technically, PPML does not assume the data is Poisson-distributed. It only requires correct specification of the conditional mean — making it a quasi-maximum likelihood estimator."
  - "See Correia, Guimarães, and Zylkin (2020), \"Fast Poisson estimation with high-dimensional fixed effects,\" <em>The Stata Journal</em>, 20(1), 95–115."
  - "Data from the <code>gravity_zeros</code> dataset in the R <code>gravity</code> package (Pacha, 2019). Cross-sectional bilateral trade flows for 2006, 22,588 observations. Available via <code>install.packages('gravity')</code>."
  - "Data from the <code>NMES1988</code> dataset in the R <code>AER</code> package (Kleiber and Zeileis). National Medical Expenditure Survey, 4,406 observations. Available via <code>install.packages('AER')</code>."

---
Throughout my time at UBC I've been lucky to have been able to explore research from not only economics but from most of the quantitative social sciences. And something that I have noticed when reviewing papers for the undergraduate journal and in conversations with friends and peers who have been enthusiastically writing their capstone thesis projects, is the sort of general fascination with using the $log(X+c)$ transformation on the regressand. As someone who's first rigourous experience doing research was in trade, where the literature almost always has used PPML (Poisson Pseudo-Maximum Likelihood), I found this to be striking. And in my conversations with a friend, he mentioned that the LLM Claude had infact recommended that he indeed use the log plus c transformation which I found very surprising. I thought why not use this as motivation to write a short note on the problems with log plus c. To the seasoned eye most of what you are going to read may may seem to be a dumbed down verios of one of the classics of Trade Literature "The Log of Gravity" by Santos Silva and Tenreyro (2006) but my vision for this is to help undergrads like me understand why one would be better of using PPML and not just in trade but in most empirical settings.

## So Why Do We Log in the First Place
{: #why-we-log}

Taking the natural log of a variable is one of the most common transformations in applied economics and we do it for good reasons. Many economic relationships are multiplicative, not additive for example a 10% tariff reduction does not add a fixed dollar amount to trade flows rather it scales them proportionally. Log-linear models happen to capture this effect naturally. A regression of log(Y) on X gives us coefficients that approximate elasticities where a one-unit change in X is associated with a roughly $\beta$ percent change in Y.

This works beautifully when the data cooperates. But the natural log has a well-known limitation that is evident to whose mathematical sophistication is bounded by introductory calculus i.e.; log(0) is undefined.

## The Zeros Problem
{: #the-zeros-problem}

In many empirical settings in trade, patents, foreign direct investment, bilateral aid, or any such count data, zeros are not just common but economically meaningful. Two countries might not trade with each other at all. A firm might file zero patents in a given year. A region might receive no foreign investment.

It would be naive of someone to interpret this as missing data. They are real observations that carry information. Dropping them biases your sample toward larger, more active observations. It is the equivalent of studying income inequality by only looking at people who earn a positive wage.

So researchers face a dilemma. They want the log transformation for its interpretive convenience and its ability to handle skewed distributions. But they cannot take the log of zero. The most common workaround is to add a small constant — typically 1 — before taking the log:

$log(Y + 1)$

This "solves" the zero problem in the sense that the code runs. But it introduces a different problem that is far less visible and, in many cases, far more damaging.

## What $log(X+1)$ Actually Does
{: #what-log-x-plus-1-actually-does}

The transformation log(X + 1) is not a neutral fix. It fundamentally changes the relationship between the variable and the outcome you are modelling.<a href="#fn-2" id="fnref-2" class="footnote-ref">2</a>

Consider two observations. Country A exports $1,000 worth of goods. Country B exports $1,000,000. Without the transformation:

- $log(1000) \approx 6.91$
- $log(1000000) \approx 13.82$

The ratio of logged values is about 2:1, reflecting the proportional relationship. Now apply log(X + 1):

- $log(1001) \approx 6.91$
- $log(1000001) \approx 13.82$

For large values, the +1 is negligible. But for small values and zeros, it dominates:

- $log(0 + 1) = log(1) = 0$
- $log(1 + 1) = log(2) \approx 0.69$
- $log(10 + 1) \approx 2.40$

The transformation compresses small values toward zero in a highly nonlinear way. An observation of 0 and an observation of 1 — which might represent very different economic realities — are mapped to 0 and 0.69 respectively. Meanwhile, the distance between 1,000 and 10,000 in log space barely changes whether you add 1 or not.

The +1 is doing real work at the bottom of the distribution and almost nothing at the top. This is not a property of the data. It is an artefact of your transformation.

## The Elasticity Breaks
{: #the-elasticity-breaks}

The whole reason we like to use the log-linear model is because of how easily interprable elasticities are. You run $\log(Y) = \alpha + \beta X + \varepsilon$ and your $\beta$ tells you the approximate percentage change in $Y$ for a one-unit change in $X$. That interpretation comes from basic calculus:

$\frac{d(\log Y)}{dX} = \frac{1}{Y} \cdot \frac{dY}{dX}$

which gives us the elasticity. Simple and elegant.

But when you estimate $\log(Y + 1) = \alpha + \beta X + \varepsilon$, that interpretation quietly falls apart.<a href="#fn-2" id="fnref-2" class="footnote-ref">2</a> The marginal effect now depends on the level of $Y$:

$\frac{d(\log(Y+1))}{dX} = \frac{1}{Y+1} \cdot \frac{dY}{dX}$

For large $Y$, the $(Y+1)$ is basically $Y$ and you're fine. But for small $Y$ — which are precisely the observations that made you add the $+1$ in the first place — the denominator is dominated by the constant. When $Y = 0$, the marginal effect is entirely an artefact of the $+1$. It has nothing to do with the underlying data-generating process.

And this bias is not random. It's systematic. It compresses variation among small observations and inflates the apparent importance of the distinction between zero and positive. If you're running a gravity model, this means the intensive margin (how much countries trade) and the extensive margin (whether they trade at all) get muddled together in ways your model simply cannot disentangle.

To make matters worse, the choice of constant is completely arbitrary. $\log(Y + 0.01)$, $\log(Y + 0.5)$, and $\log(Y + 1)$ all give you different coefficient estimates. There is no principled reason to pick one over the other, and yet the results can be very sensitive to this choice, especially when zeros make up a large share of your sample.<a href="#fn-1" id="fnref-1" class="footnote-ref">1</a>

## PPMLHDFE: The Right Tool
{: #ppmlhdfe}

So what should you use instead? This is where Poisson Pseudo-Maximum Likelihood (PPML) comes in and specifically its implementation with high-dimensional fixed effects: PPMLHDFE.<a href="#fn-3" id="fnref-3" class="footnote-ref">3</a>

The key idea is that PPML estimates the model in levels, not in logs. Instead of transforming your dependent variable, you specify:

$\mathbb{E}[Y \mid X] = \exp(X\beta)$

$Y$ stays in its natural units. The exponential form captures the multiplicative structure that motivated taking logs in the first place but because you're estimating in levels, zeros are handled naturally since $\exp(X\beta)$ can predict any non-negative value including values close to zero.

What makes PPML especially appealing to me and what I think should convince anyone who's been defaulting to $\log(Y+1)$ comes down to three things:

First, **it handles zeros without any transformation.** No arbitrary constants. No dropping observations. Zeros enter the estimation as what they are: real data points with real informational content.

Second, **the coefficients actually mean what you want them to mean.** $\beta$ in a PPML model represents the semi-elasticity of $Y$ with respect to $X$, exactly as in a properly specified log-linear model but without the distortion that the $+1$ introduces.

Third, and this is the one that I think gets underappreciated, **it's robust to heteroskedasticity.** Santos Silva and Tenreyro (2006) showed that OLS estimation of log-linearised models is actually inconsistent when your errors are heteroskedastic which, let's be honest, is basically always the case in economic data.<a href="#fn-1" id="fnref-1" class="footnote-ref">1</a> PPML only requires correct specification of the conditional mean, not the full distribution.<a href="#fn-4" id="fnref-4" class="footnote-ref">4</a>

The PPMLHDFE implementation makes all of this practical. It can absorb the high-dimensional fixed effects that are standard in modern empirical work — country-pair, country-year, industry-year — without creating thousands of dummy variables.<a href="#fn-5" id="fnref-5" class="footnote-ref">5</a>

And here's the thing that really gets me: the syntax is almost identical. In Stata:

```
ppmlhdfe trade tariff, absorb(pair_id year_id) cluster(pair_id)
```

Compare that to the $\log(Y+1)$ approach:

```
reghdfe log_trade_plus1 tariff, absorb(pair_id year_id) cluster(pair_id)
```

You're changing one command. That's it. The cost of switching is essentially zero but the difference in what you get out is substantial.

## Seeing It in Trade Data
{: #a-live-example}

I don't think any of this is fully convincing without seeing it on real data so let me show you what this looks like in practice.<a href="#fn-6" id="fnref-6" class="footnote-ref">6</a>

The `gravity_zeros` dataset in R has 22,588 bilateral trade observations for 2006. Of these, 5,500 — roughly 24% — are zeros. This is a pretty standard gravity setting: bilateral trade flows regressed on distance, contiguity, common language, and regional trade agreements with exporter and importer fixed effects.

I ran three specifications on this data:

1. **OLS on $\log(Y)$** — dropping all 5,500 zeros (17,088 observations)
2. **OLS on $\log(Y+1)$** — keeping zeros via the $+1$ transformation (22,588 observations)
3. **PPML** — keeping zeros, estimating in levels (22,588 observations)

Here's what you get:

|  | OLS $\log(Y)$ | OLS $\log(Y+1)$ | PPML |
|---|---:|---:|---:|
| $\log(\text{Distance})$ | $-1.617$ | $-0.807$ | $-0.823$ |
|  | $(0.033)$ | $(0.017)$ | $(0.037)$ |
| Contiguity | $0.918$ | $0.970$ | $0.395$ |
|  | $(0.113)$ | $(0.077)$ | $(0.063)$ |
| Common Language | $0.992$ | $0.458$ | $0.240$ |
|  | $(0.059)$ | $(0.028)$ | $(0.061)$ |
| RTA | $0.498$ | $0.837$ | $0.421$ |
|  | $(0.062)$ | $(0.037)$ | $(0.078)$ |
| *Observations* | *17,088* | *22,588* | *22,588* |
| *Zeros included* | *No* | *Yes* | *Yes* |

<div class="figure">
  <img src="/assets/images/ppml-comparison.svg" alt="Coefficient comparison: OLS vs PPML" class="light-only">
  <img src="/assets/images/ppml-comparison-dark.svg" alt="Coefficient comparison: OLS vs PPML" class="dark-only">
  <p class="figure-caption">Coefficient estimates with 95% confidence intervals. The log(Y+1) specification (red) systematically inflates estimates relative to PPML (green), particularly for binary variables like contiguity, common language, and RTA membership.</p>
</div>

Look at those numbers. Compared to PPML, the $\log(Y+1)$ specification overstates the contiguity effect by 145%, the common language effect by 91%, and the RTA effect by 99%. These aren't rounding differences. If you were writing a policy memo based on the $\log(Y+1)$ results you would be telling someone that sharing a border matters more than twice as much as it actually does.

Notice also that even the OLS on $\log(Y)$ — which drops zeros entirely — gives different results from $\log(Y+1)$. Neither OLS approach matches PPML. The $\log(Y+1)$ transformation doesn't "fix" the zeros problem; it trades one source of bias (sample selection) for another (functional form distortion) and honestly the second one can be worse because at least when you drop zeros you know you're losing information. With $\log(Y+1)$ the bias is invisible.

If you want to replicate this yourself it's straightforward:

```r
library(gravity)
library(fixest)

data(gravity_zeros)
df <- gravity_zeros
df$ln_dist <- log(df$distw)
df$ln_flow_plus1 <- log(df$flow + 1)
df$ln_flow <- ifelse(df$flow > 0, log(df$flow), NA)

# OLS dropping zeros
feols(ln_flow ~ ln_dist + contig + comlang_off + rta |
      iso_o + iso_d, data = df, vcov = "hetero")

# OLS with log(Y+1)
feols(ln_flow_plus1 ~ ln_dist + contig + comlang_off + rta |
      iso_o + iso_d, data = df, vcov = "hetero")

# PPML
fepois(flow ~ ln_dist + contig + comlang_off + rta |
       iso_o + iso_d, data = df, vcov = "hetero")
```

## Beyond Trade: ER Visits with 82% Zeros
{: #beyond-trade}

Now you might be thinking: this is a trade problem. Gravity models are a specific setting, PPML was developed by trade economists, and maybe this doesn't apply to what I'm working on. That was exactly my reaction when I first learned about it. But it does, and I want to show you why.<a href="#fn-7" id="fnref-7" class="footnote-ref">7</a>

The `NMES1988` dataset from the National Medical Expenditure Survey has 4,406 observations on healthcare utilization. The outcome I'll focus on is emergency room visits. Of the 4,406 individuals in the sample, 3,602 — a staggering 82% — had zero ER visits. This is a setting that has absolutely nothing to do with trade, but the zeros problem is identical.

I ran the same three specifications: OLS on $\log(Y)$ dropping zeros, OLS on $\log(Y+1)$ keeping zeros, and PPML. The regressors are self-reported health status, number of chronic conditions, age, income, insurance status, Medicaid coverage, and gender.

|  | OLS $\log(Y)$ | OLS $\log(Y+1)$ | PPML |
|---|---:|---:|---:|
| Health: Poor | $0.132$ | $0.123$ | $0.635$ |
|  | $(0.043)$ | $(0.022)$ | $(0.099)$ |
| Health: Excellent | $0.048$ | $-0.049$ | $-0.642$ |
|  | $(0.068)$ | $(0.013)$ | $(0.215)$ |
| Chronic Conditions | $0.049$ | $0.038$ | $0.241$ |
|  | $(0.012)$ | $(0.005)$ | $(0.028)$ |
| Medicaid | $0.016$ | $0.052$ | $0.252$ |
|  | $(0.056)$ | $(0.024)$ | $(0.129)$ |
| *Observations* | *804* | *4,403* | *4,403* |
| *Zeros included* | *No* | *Yes* | *Yes* |

<div class="figure">
  <img src="/assets/images/ppml-healthcare.svg" alt="Coefficient comparison: ER visits" class="light-only">
  <img src="/assets/images/ppml-healthcare-dark.svg" alt="Coefficient comparison: ER visits" class="dark-only">
  <p class="figure-caption">ER visit coefficients with 95% confidence intervals. With 82% zeros, the log(Y+1) specification (red) massively understates effects compared to PPML (green).</p>
</div>

The results here are arguably even more alarming than the trade example. Look at what happens to the "Health: Poor" coefficient. PPML says being in poor health is associated with a 63.5% increase in ER visits — which makes intuitive sense. But log(Y+1) gives you 0.123, understating the effect by over 80%. For chronic conditions the story is similar: PPML estimates a 24.1% increase per additional chronic condition, while log(Y+1) gives you 3.8%.

And here's the really telling part: OLS on $\log(Y)$ — which drops 82% of the sample — gives you only 804 observations. You're literally throwing away most of your data. And even then the coefficients don't match PPML because the remaining sample is severely selected toward sicker individuals.

The direction of bias actually flipped compared to the trade example. In gravity data with 24% zeros, log(Y+1) overstated effects. In healthcare data with 82% zeros, it understates them. The sign and magnitude of the bias depends on the distribution of zeros in your specific data — which makes it impossible to "correct for" without simply using the right estimator.

This is the point I really want to drive home. PPML is not a trade tool. It's a count data tool. Anytime your dependent variable is non-negative with a mass of zeros — ER visits, patent filings, aid disbursements, conflict events, firm entry counts, citations — the same logic applies. The $\log(Y+1)$ specification will distort your estimates, and the distortion will be worse the more zeros you have.

## When This Matters
{: #when-this-matters}

To be clear, not every dataset has a zero problem. If your dependent variable is always positive — log wages, log GDP, log prices — the log transformation works perfectly and you don't need PPML. I'm not arguing that we should never take logs.

But if you're working with trade flows where many country-pairs simply don't trade, or patent counts where most firms file zero, or FDI, aid flows, event counts of conflicts or natural disasters — basically any setting where zeros are a meaningful part of your data — then $\log(Y+1)$ is not the innocent convenience it appears to be. It's a specification choice that distorts your estimates in ways that are hard to diagnose and impossible to fix after the fact.

The fix is straightforward: use PPMLHDFE. You get the multiplicative model you wanted in the first place, zeros are handled properly, estimates are consistent under heteroskedasticity, and it can handle whatever fixed-effect structure your research design demands. There is genuinely very little reason to reach for $\log(Y+1)$ when this tool is sitting right there.

---

I think the broader point is simple. Econometric convenience should not come at the cost of econometric integrity. The $\log(Y+1)$ transformation persists because it's easy to implement and because its problems are invisible in a regression table. The coefficients look reasonable. The standard errors are well-behaved. Nothing flags the bias. But the bias is there, baked into the functional form, varying with the level of $Y$, and sensitive to an arbitrary constant that you chose for no principled reason.

PPML is not a new method — Santos Silva and Tenreyro published "The Log of Gravity" nearly twenty years ago. PPMLHDFE has been available in Stata since 2019 and in R via `fixest` for just as long. The tools exist. The theory is settled. If you're an undergrad working on your thesis and someone — whether it's a friend, a textbook, or an LLM — tells you to just add 1 and take the log, I hope this gives you a reason to push back.
