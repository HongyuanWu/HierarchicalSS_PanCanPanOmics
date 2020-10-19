# A Hierarchical Spike-and-Slab Model for Pan-Cancer Survival Using Pan-Omic Data

This is the repository for code used for the project titled "A Hierarchical Spike-and-Slab Model for Pan-Cancer Survival Using Pan-Omic Data." The purpose of this project was to develop methodology for prediction using multiple sources of data and multiple sample sets. We used results of a method for bidimensional integration of multi-source, multi-way data called BIDIFAC+ by Lock et al. (https://arxiv.org/abs/2002.02601) to predict overall patient survival from the Cancer Genome Atlas database. Our predictive model is a Bayesian hierarchical model with hierarchical spike-and-slab predictors that allow for the borrowing of information across sample sets when determining the sparsity structure of the data. This repository also provides simulation results for this hierarchical model under different data-generating conditions. 

The following are instructions on how to reproduce our results. 

## Reading in and processing the data

`load_modules.R` contains code for loading in the BIDIFAC+ modules and computing their SVDs. The scores that we selected based on our filtering criteria will be saved at the end. Download `pan.fac.results_v2.rda` and edit all `load()` and `save()` commands to your directory. This script will save a file called `mod.pcs.v3.rda` which contains the selected features from the BIDIFAC+ results. These features are in a variable named `mod.pcs`.

`MatchingClinicalAndFactorizationData.R` matches the features of the BIDIFAC+ module SVDs to TCGA clinical data. You will have to change all `load()` statements to reflect your directory. In the end, this script will save a file called `XYC_V2_WithAge_StandardizedPredictors.rda` which contains the features, the survival data, and the censored data for all the subjects. 

`CheckingDataMatchingResults.R` checks proper matching of subjects from modules to TCGA clinical data. This ensures data is of proper form (no missing, positive survival times, etc). This script begins by deconstructing the matching function from the previous script and then performs an explicit checker to ensure the data matches properly. In order to properly check that the age column matches, you will have to change the following line out of the `MatchingClinicalAndFactorizationData.R`script from:

```
age.centered.i = (X_i[, "0.5"] - mean(X_i[, "0.5"]))/sd(X_i[, "0.5"]
```

to

```
age.centered.i = X_i[, "0.5"]
```

because the clinical data has the original age values, not the standardized version. 

The last two scripts (and most of the later ones) source `HelperFunctions.R` which contains helper functions used throughout project and analysis.

## The Gibbs sampler 

`ExtendedHierarchicalModelLogNormalSpikeAndSlabInterceptOutsideSS.R` contains code for the Gibbs sampling algorithm used in analysis. Assumes *log-normally* distributed survival and applies hierarchical spike-and-slab priors to coefficients, excluding the intercept. We use a version of this model that assumes *normally* distributed survival to check proper coverage for simplicitly. Code for the normal-likelihood model can be found in `ExtendedHierarchicalModelNormalSpikeAndSlab.R`. This model is only used for coverage checking purposes. 

The coverage simulation can be found in `ExtendedHierarchicalModelSpikeAndSlabCoverageSimulation.R` which ensures 95% coverage of credible intervals generated by Gibbs sampling algorithm. `ExtendedHierarchicalModelSpikeAndSlabSelectingTrueParametersSimulation.R` contains code for the selection accuracy simulation to ensure Gibbs sampling algorithm correctly identifies predictors to include in model. This simulation should give high accuracy, around 90%. 

## TCGA Data Application

`ModelComparison.R` compares 5 different possible models for best fit on TCGA data. We conclude the hierarchical spike-and-slab model we propose fits the data best so we apply it to the TCGA data in `GibbsSamplingResults_NewFactorizationDataFilterV3_WithAge_StandardizedPredictors.R`. This script runs the Gibbs sampling algorithm on the TCGA data for the log-normal model. Results are saved for further analysis. This script also creates inclusion heatmap we include in our article.

## Model Simulations

We run our model simulations to compare the performance of our model with four other Bayesian hierarchical frameworks in `ModelSimulations.R`. We compare these five models under six different data-generating scenarios to assess the performance and flexibility of each. This script sources `RunFiveModelTypes.R` which runs the five different possible models to make things easier. 

