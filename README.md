# YuleInfer

**Inference for correlated binary data using Yule's Colligation Coefficient**

YuleInfer is an R package for estimating and testing a common marginal risk difference between two groups with correlated binary outcomes. It supports paired observations, such as outcomes from two eyes or two ears, together with unpaired observations.

Analyses can involve one stratum or multiple strata. The package provides a unified workflow under four association models: **Yule, Dallal, Rosner and Donner**.

## Features

- Common marginal risk-difference estimation.
- Likelihood-ratio, efficient-score and Wald tests.
- LR and score confidence sets obtained by test inversion, and Wald confidence intervals.
- Mixed bilateral and unilateral data, including bilateral-only and unilateral-only analyses.
- Subject-level data import and aggregated count input.
- Explicit treatment ordering and interpretable analysis summaries.
- Convergence, boundary and confidence-set completeness diagnostics.
- Simulation utilities, documented examples and a published clinical dataset.

## Installation

Download `YuleInfer_0.5.2.tar.gz` from this repository and place it in your R working directory. Then run:

```r
install.packages(
  "YuleInfer_0.5.2.tar.gz",
  repos = NULL,
  type = "source"
)

library(YuleInfer)
```

**Requirements:** R version 4.0 or later. Installing this source package requires a C compiler configured for R, including compatible Rtools on Windows.

Runtime dependencies are limited to R's standard `stats` and `utils` packages. Rendered documentation and vignettes are included in the source archive.

## Quick start: published otitis media data

The package includes `otitis_media`, an aggregated clinical dataset comparing cefaclor and amoxicillin. It contains 203 evaluable children across three age strata, including 75 bilateral and 128 unilateral subjects.

```r
library(YuleInfer)

data(otitis_media)

# Define the reference treatment explicitly.
dat <- strat_data(
  otitis_media,
  reference = "Amoxicillin"
)

# Fit the Yule model, test a zero risk difference,
# and construct a 95% LR confidence interval.
result <- yule_analyze(
  dat,
  model = "yule",
  d0 = 0,
  method = "LR",
  conf.level = 0.95
)

result
```

The risk difference is defined as **comparison group minus reference group**. In this example, it is the probability of an effusion-free ear under cefaclor minus that under amoxicillin.

The estimated risk difference is approximately **0.1689**, with a **95% LR confidence interval of 0.0422–0.2922** and a two-sided **p-value of 0.0092**.

See `?otitis_media` for the original publications, response coding and data provenance.

## Compare the four association models

The same data format and inference functions are used for every model.

```r
models <- c("yule", "dallal", "rosner", "donner")

analyses <- setNames(
  lapply(models, function(m) {
    yule_analyze(dat, model = m, method = "LR")
  }),
  models
)

comparison <- do.call(
  rbind,
  lapply(analyses, as.data.frame)
)

comparison
```

Within each stratum, the selected association parameter is shared between the two treatment groups. Association parameters may differ across strata.

The model choices impose different assumptions. Comparing their results does not establish that those assumptions hold.

## Choose a test or confidence interval

Supported inference methods are `"LR"`, `"score"` and `"Wald"`. All tests are two-sided.

A fitted model can be reused:

```r
fit <- result$fit

# Efficient-score test of a zero common risk difference.
strat_test(
  fit,
  d0 = 0,
  method = "score"
)

# A 95% Wald confidence interval.
strat_confint(
  fit,
  method = "Wald",
  level = 0.95
)
```

Use `conf.level` with `yule_analyze()` and `level` with `strat_confint()`.

## Data format

For aggregated data, provide one row for each treatment group within each stratum.

| Column | Meaning |
|---|---|
| `stratum` | Stratum identifier; optional for a single stratum |
| `group` | Treatment-group label |
| `m0` | Bilateral subjects with zero positive responses |
| `m1` | Bilateral subjects with one positive response |
| `m2` | Bilateral subjects with two positive responses |
| `u0` | Unilateral subjects with a negative response |
| `u1` | Unilateral subjects with a positive response |

Counts refer to **subjects**. A bilateral subject contributes once to the sample size.

For subject-level records, use `yule_import()` to prepare the data. See `?yule_import` for examples.

## Assumptions and diagnostics

The methods assume independent subjects, exchangeable paired responses, a common marginal risk difference across strata, and the selected shared-association restriction within each stratum. Bilateral and unilateral subjects in the same treatment–stratum cell are assumed to have the same per-unit marginal response probability.

Inference uses large-sample approximations. Sparse data, boundary estimates or unsuccessful numerical searches can produce unavailable tests or incomplete confidence sets.

Inspect the reported diagnostics:

```r
result$diagnostics
result$confint
```

LR and score inversion can produce disconnected confidence sets. The package retains detected components and flags unresolved regions. Unavailable tests should not be counted as nonrejections, and incomplete confidence sets should not be treated as ordinary intervals.

## Documentation

```r
help(package = "YuleInfer")

vignette("getting-started", package = "YuleInfer")
vignette("models-and-inference", package = "YuleInfer")

browseVignettes("YuleInfer")
```

A complete clinical example covering all four models is included:

```r
source(
  system.file(
    "examples",
    "published_ome.R",
    package = "YuleInfer"
  )
)
```

## Citation

To obtain the package citation and BibTeX entry:

```r
citation("YuleInfer")
```

Please also cite the original data publications when using `otitis_media`; these are listed in its help page.

## Authors and contact

**Authors:** Guanjie Lyu and Xinwei Huang.

**Maintainer:** Guanjie Lyu  
**Email:** lvg@uwindsor.ca

## License

MIT license. See `LICENSE` and `LICENSE.md` in the source package.
