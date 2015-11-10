
![http://quantnet.wiwi.hu-berlin.de/style/banner.png](http://quantnet.wiwi.hu-berlin.de/style/banner.png)

# ![qlogo](http://quantnet.wiwi.hu-berlin.de/graphics/quantlogo.png) **TSARMARJMCMC**


## Full description
__Reversible Jump Markov Chain Monte Carlo (RJMCMC)__ Sampler for __Autoregressive Moving Average (ARMA)__ Time Series Models

```yaml

Name of Quantlet: TSARMARJMCMC

Published in: Univariate Time Series

Description: 'Estimate univariate, stationary Autoregressive Moving 
              Average (ARMA) models using Reversible Jump Markov Chain 
              Monte Carlo'

Keywords: Kalman filter, Markov, PACF, arma, stationary, univariate

Author: Daniel Neuhoff, Alexander Meyer-Gohde

Submitted: Mon, October 12 2015 by neuhoffd

Input: Any zero-mean stationary univariate time series

Output: 'Posterior distribution of parameters and orders of lag polynomials
         Mean and median estimates for parameters'

```

## 1. Purpose
This quantlet contains a small suite enabling the user to estimate ARMA time series models using Reversible Jump Markov Chain Monte Carlo (RJMCMC)
(See e.g. Green 1995, Brooks and Ehlers 2003). RJMCMC enables the sampling from posteriors over not only the parameter space for a particular model,
but also several models.


## 2. Scientific details
The sampler provided here assumes zero-mean stationary ARMA models with normal disturbances as in Neuhoff (2015) or Meyer-Gohde and Neuhoff (2015).
Stationarity is ensured by reparametrizing the lag polynomials in terms of (inverse) partial autocorrelations as described in Monahan 1984.
Thus, priors and proposals have to be supplied in these terms. The supplied code evaluates the likelihood by means of the Kalman filter.


## 3. Technical details
The sampler settings and data source can be set in the file 
[getSettings.m] (https://github.com/QuantLet/RJMCMC/blob/master/getSettings.m).
It is also possible to replace all prior distributions,
proposal distributions, as well as the Likelihood functions via setting the appropriate function handles in this file.
This framework can thus be also employed to estimate ARMA models with non-normal disturbances.
Furthermore, any proposal distribution can be used. For further information please refer to the comments in "getSettings.m".


## 4. Getting started
In order to run the sampler, set the desired options in "getSettings.m" and run "estimateARMA.m". To display the results, run "displayResults.m".
Depending on the value of the variable settings.doPlots to be set in "getSettings.m", this script will also plot the conditional
and unconditional posterior averages for the parameters. Some synthetic data for testing is provided in testdata.mat.
The data was generated from an AR(1) process with the standard deviation of the error term set to 0.9. The sample size is 250.

``` matlab
clear all; 
close all; home; format long g;  rng('shuffle');

settings = getSettings();

datum = date;
if settings.saveProposals
    global PROPOSALS_GLOBAL;
    PROPOSALS_GLOBAL = getEmptyDrawStruct();
    PROPOSALS_GLOBAL(settings.draws) = getEmptyDrawStruct();
end;

tic;
[states, accepted] = doRJMCMC(settings);
disp('Time elapsed for sampling'); toc;
        
arParametersSeries = padcat(states.arParameters);
maParametersSeries = padcat(states.maParameters);
sigmaESeries = cat(1,states.sigmaEs);
pSeries = cat(1,states.ps);
qSeries = cat(1,states.qs);
arPacsSeries = padcat(states.arPacs);
maPacsSeries = padcat(states.maPacs);
logPosteriorSeries = cat(1,states.logPosterior);
        
if settings.saveProposals
    arParametersSeries_PROPOSAL = padcat(PROPOSALS_GLOBAL.arParameters);
    maParametersSeries_PROPOSAL = padcat(PROPOSALS_GLOBAL.maParameters);
    sigmaESeries_PROPOSAL = cat(1,PROPOSALS_GLOBAL.sigmaEs);
    pSeries_PROPOSAL = cat(1,PROPOSALS_GLOBAL.ps);
    qSeries_PROPOSAL = cat(1,PROPOSALS_GLOBAL.qs);
    arPacsSeries_PROPOSAL = padcat(PROPOSALS_GLOBAL.arPacs);
    maPacsSeries_PROPOSAL = padcat(PROPOSALS_GLOBAL.maPacs);
end;

if settings.saveProposals
    save(settings.resultsFile,'arParametersSeries', 'maParametersSeries', 'sigmaESeries', 'pSeries', 'qSeries', 'arPacsSeries','maPacsSeries',...
        'arParametersSeries_PROPOSAL', 'maParametersSeries_PROPOSAL', 'sigmaESeries_PROPOSAL', 'pSeries_PROPOSAL', 'qSeries_PROPOSAL', 'arPacsSeries_PROPOSAL','maPacsSeries_PROPOSAL',...
        'logPosteriorSeries', 'accepted', 'settings');
else            
    save(settings.resultsFile,'arParametersSeries', 'maParametersSeries', 'sigmaESeries', 'pSeries', 'qSeries', 'arPacsSeries',...
    'maPacsSeries', 'logPosteriorSeries', 'accepted',  'settings');
end;  

displayResults;
```

## 5. Simulation results
### The plots are displayed as follows:

* Posterior distribution of lag polynomial orders

![Posterior for lag polynomial orders] (SamplePosteriorOrders.png)

* Recursive means for parameters

![Recursive mean for parameters] (ConditionalMeanARParameterSample.png)

* Prior posterior plots

![Prior Posterior Plot] (PriorCondPosteriorARPAC1FILTERED.png)


## 6. Appendix 
To install the sampler, you can download all necessary files by clicking on "Download ZIP" on the right. Extract the files to a directory
of your choosing, navigate there within Matlab or add it to the Matlab path, set the preferences in getSettings.m to your liking
and run estimateARMA.m.

Tested on Matlab R2015a running on Windows Server 2014 and Windows 7 Professional. Utilizes the Statistics Toolbox.
Note that the sampler requires significant amounts of RAM depending on the number of draws. At least 8 GB are recommended!
