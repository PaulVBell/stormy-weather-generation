# Stormy Weather Generation

## Overview

This project aims to explore evaluation methods for a Bayesian weather generator using two key methods taken from Gneiting et al. (2007):

1. Proper scoring rules 
2. Graphical tests for model calibration.

Gneiting, Tilmann, Fadoua Balabdaoui, and Adrian E. Raftery. ‘Probabilistic Forecasts, Calibration and Sharpness’. Journal of the Royal Statistical Society: Series B (Statistical Methodology) 69, no. 2 (2007): 243–68. https://doi.org/10.1111/j.1467-9868.2007.00587.x.

## Data

The following data comes from 3 sources.

1. Atmospheric data is from the European Centre for Medium-Range Weather Forecasts (ECMWF) Reanalysis version 5 (ERA5) (Hersbach et al. 2023).
2. MIDAS Open UK hourly rainfall data, v202308 (Met Office 2023).
3.  European Storm Types Dataset (Catto and Dowdy 2023).

Hersbach, H., Bell, B., Berrisford, P., Biavati, G., Horányi, A., Muñoz Sabater, J., Nicolas, J., Peubey, C., Radu, R., Rozum, I., Schepers, D., Simmons, A., Soci, C., Dee, D., Thépaut, J-N. (2023): ERA5 hourly data on pressure levels from 1940 to present. Copernicus Climate Change Service (C3S) Climate Data Store (CDS), DOI: <https://doi.org/10.24381/cds.bd0915c6> (Accessed on 16-12-2024)

Met Office (2023): MIDAS Open: UK hourly rainfall data, v202308. NERC EDS Centre for Environmental Data Analysis, 03 October 2023. doi: <https://doi.org/10.5285/c21639861fb54623a749e502ebac74ed>.

Catto, J., Dowdy, A. European Storm Type Dataset (2023) DOI: <https://doi.org/10.24378/exe.4764>

## Methods

We use Bayesian Generalised Linear Models (GLMs):

Let $r_t$ be the amount of precipitation in a 6 hour time step $t$, where any precipitation amount less than or equal to the threshold of occurrence, $r_{\text{th}} = 1 \text{ mm 6hr}^{-1}$, is considered dry and set to zero.

Then let $o_t$ be an indicator variable for the occurrence of precipitation, so $o_t$ is 0 (1) if $r_t \le 1 \text{ mm 6hr}^{-1}$ ($r_t > 1 \text{ mm 6hr}^{-1}$).

Finally, let $y_t$ be the amount of precipitation greater than $r_{\text{th}}$ in a 6 hour time step $t$ given the occurrence of precipitation, i.e. $o_t = 1$.
The model is then specified as follows:
$$
    o_t | p_t \sim \text{Bernoulli}(p_t),
$$
$$
    y_t | \alpha, \beta_t \sim\text{Gamma}(\alpha,\beta_t),
$$
$$
    \alpha/\beta_t = \mu_t,
$$

$$
    r_t = o_t(y_t + r_{\text{th}}).
$$

The quantities $p_t$ and $\mu_t$ are related to a linear combination of covariate values via a logit and logarithmic link function respectively:

$$
g(p_t) = \text{log} \left( \frac{p_t}{1-p_t} \right) = \bm{x}_{t,o}^T \bm{\theta}^{(s)}_o,
$$

$$
g(\mu_t) = \text{log}(\mu_t) = \bm{x}_{t,a}^T \bm{\theta}^{(s)}_a.
$$

A different regression parameter is modeled for each storm type (superscript $^{(s)}$).

This is a Bayesian framework so parameters $\bm{\theta}^{(s)}_o$, $\bm{\theta}^{(s)}_a$ and $\alpha$ have prior distributions:

$$
\theta^{(s)}_i \sim \text{Normal}(0, \sigma_i), 
$$

$$
\alpha \sim \text{Gamma}(2,1).
$$

A different $\sigma_i$ is chosen for each regression parameter, based on the prior predictive distribution (not included in this vignette).

## Installation

The weather generator vignette requires the following R packages:

```
install.packages(c("brms","tidyverse","reshape2","future"))
```

Note that brms is based on Stan and therefore requires the installation of a C++ compiler, e.g. Rtools (https://cran.r-project.org/bin/windows/Rtools/) for Windows. See https://github.com/stan-dev/rstan/wiki/RStan-Getting-Started for more detail.

Note that there is a 1.6 GB .RData file in the fits folder.
