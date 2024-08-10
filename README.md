
<!-- README.md is generated from README.Rmd. Please edit that file -->

# asgl <img src="figures/logo.png" align="right" height="150" alt="funq website" /></a>

[![Downloads](https://pepy.tech/badge/asgl)](https://pepy.tech/project/asgl)
[![Downloads](https://pepy.tech/badge/asgl/month)](https://pepy.tech/project/asgl)
[![License: GPL
v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Package
Version](https://img.shields.io/badge/version-1.0.6-blue.svg)](https://cran.r-project.org/package=asgl)

<style>
body {
text-align: justify}
</style>

## Introduction

`asgl` is a Python package that solves several regression related models
for simultaneous variable selection and prediction, in low and high
dimensional frameworks. This package is directly related to research
work shown on [this
paper](https://link.springer.com/article/10.1007/s11634-020-00413-8) and
a full description of the capabilities of the package is shown on [this
paper](https://arxiv.org/abs/2111.00472). We also suggest accessing the
user_guide notebook provided in the [GitHub
repository](https://github.com/alvaromc317/asgl). Or you can find more
accesible explanations
[here](https://towardsdatascience.com/sparse-group-lasso-in-python-255e379ab892)
and
[here](https://towardsdatascience.com/an-adaptive-lasso-63afca54b80d).

The current version of the package supports:

- Linear regression models
- Quantile regression models

And considers the following penalizations for variable selection:

- No penalized models
- lasso
- group lasso
- sparse group lasso
- adaptive lasso
- adaptive group lassso
- adaptive sparse group lasso

## Requirements

The package makes use of some basic functions from `scikit-learn` and
`numpy`, and is built on top of the wonderful `cvxpy` convex
optimization module. It is higly encouraged to install `cvxpy` prior of
the installation of `asgl` following the instructions from the original
authors, that can be found [here](https://www.cvxpy.org/)).
Additionally, `asgl` makes use of python `multiprocessing` module,
allowing, if requested, for parallel execution of the code highly
reducing computation time.

## Usage example:

Even though the `asgl` package can easily deal with much more complex
datasets, we will work using a synthetic dataset not affected by
computation time. We will show first how to estimate a set of adaptive
weights and fit the model using `asgl`. The following code estimates a
set of adaptive weights using a PCA approach that has been shown to
provide very good results in a general set of simulations and real data
analysis. This approach is denoted as `'pca_pct'`. Then we estimate an
linear regression model with an adaptive sparse group lasso penalization
on a grid of possible `lambda1` values (2 values), `alpha` values (3
values) and weight values (3 values for the lasso weights and 1 value
for the group lasso weight). You can check that the length of each of
the parameters array is respectively 2, 3, 3 and 1. The estimated
coefficients will be stored in the `coef_` object of the class.

``` python
# Import required packages
import numpy as np
import asgl
from sklearn.datasets import make_regression

x, y = make_regression(n_samples=1000, n_features=20, random_state=42)
group_index = np.random.randint(1, 6, 20)

# Define parameters grid
lambda1 = np.array([1e-2, 1])
alpha = np.array([0, 0.5, 1])

# Estimate the adaptive weights using the pca_pct approach
weights = asgl.WEIGHTS(weight_technique='pca_pct', lasso_power_weight=[0.8, 1, 1.2])
lasso_weights, group_lasso_weights = weights.fit(x, y, group_index=group_index)

# Fit the adaptive sparse group lasso model on a grid of possible lambda1, alpha and weight values.
model = asgl.ASGL(model='lm', penalization='asgl', lambda1=lambda1, alpha=alpha,
                  lasso_weights=lasso_weights, gl_weights=group_lasso_weights)
```

The package also implements cross validation capabilities. The next
example shows how to use cross validation to find the optimal parameter
values in the previous dataset and how to compute the test error.

``` python
# Define a cross validation object
cv_class = asgl.CV(model='lm', penalization='asgl', weight_technique='pca_pct', 
                   lambda1=lambda1, alpha=alpha, nfolds=3, error_type='MSE', 
                   parallel=True)

# Compute error using k-fold cross validation. 
# Error is a matrix of shape (number_of_models, k_folds)
error = cv_class.cross_validation(x=x, y=y, group_index=group_index)

# Obtain the mean error across different folds
error = np.mean(error, axis=1)

# Select the minimum error
minimum_error_idx = np.argmin(error)

# Select the parameters associated to mininum error values
optimal_parameters = cv_class.retrieve_parameters_value(minimum_error_idx)
```

    #> {'lambda1': np.float64(1.0), 'alpha': np.float64(1.0), 'lasso_weights': array([0.14037592, 0.02528164, 0.02049966, 0.05090492, 0.01602123,
    #>        0.01028269, 0.14426051, 0.0950133 , 0.04567129, 0.24991105,
    #>        0.26441192, 0.09444413, 0.04209573, 2.05322409, 0.0853902 ,
    #>        0.01110109, 0.02157117, 0.05466762, 0.03193988, 0.03812168]), 'gl_weights': array([0.01275304, 0.01024751, 0.01712996, 0.01047884, 0.01741501])}

Once obtained the set of optimal parameters, these can be used in a
`asgl` model like the one described in the first example to build the
final optimized model.

### Citing

------------------------------------------------------------------------

If you use `asgl` for academic work, we encourage you to [cite our
paper](https://link.springer.com/article/10.1007/s11634-020-00413-8).
Thank you for your support and we hope you find this package useful!
