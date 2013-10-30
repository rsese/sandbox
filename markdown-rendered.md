In this example, we will look at fitting a simple non-linear model to some data.  This is the sort of process that's often done using R's `nls` function.  Here, as usual in Bayesian statistics, we will construct an explicit probability model for our data and estimate parameter values in this model from our data.  We end up with a probability distribution for the model parameters conditional on the data (and our prior assumptions) that we can use to answer questions about our data and model.  (This example is taken from Chapter 23 of *Statistical Computing: An Introduction to Data Analysis Using S-Plus* by Michael J. Crawley.)

*As in many of these examples, there may be an impression that the amount of code you need to write makes using this modelling approach rather complex.  In fact, all of the code in this example can be generated for you by BayesHive, most of it coming from the nonlinear regression model builder.  You can use BayesHive for many statistical and modelling tasks without having to write a line of Baysig code!*


## Data

The data we're going to use records the jaw size of deer as a function of their age.  The plot below shows the raw data.

> deer = #Examples#"NL Regression Example 1"#asymptotic

?> axisLabels "Age (yr)" "Jaw size (mm)" $ scatterPlot $ with deer (x, y)


## A simple model

From the plot above, it's clear that the relationship between age and jaw size in this group of deer is not linear.  It would make no sense to attempt a linear regression between age and jaw size in this case, so we will construct a simple nonlinear model instead.

The data display asymptotic behaviour with increasing age, suggesting that we attempt to fit a model of the form

$$y = a - b e^{-cx},$$

where $x$ is age and $y$ jaw size, and $a$, $b$ and $c$ are parameters to be determined from the data.  As a probabilistic model, we write $y \sim \mathcal{N}(a - b e^{-c x}, \sigma^2)$, where $\sigma^2$ is an error variance representing the variability in the data that is not explained by our model.

A model like this can be expressed in Baysig as

> nlreg = prob
>   a ~ improper_uniform_positive
>   b ~ improper_uniform_positive
>   c ~ gamma 1 1E10
>   variance ~ gamma 1 20
>   repeat 55 $ prob
>     x ~ any
>     y ~ normal (a - b * exp(-c * x)) variance
>     return { x => x; y => y }

Note that we make explicit our assumption that the unexplained variance $\sigma^2$ is independent of $x$, and that we impose prior distributions for the parameters $a$, $b$, $c$ and $\sigma^2$.  The prior distributions we use here are completely uninformative (in terms of the parameterisation of the model we are using), except for the fact that the parameters must be positive, but it would be easy enough to use more informative priors based on reasonable seeming values for the maximum jaw size and rate of growth.


## Model fitting

We can then fit our model using Baysig's `estimate` function:

> par <* estimate nlreg deer

?> par


## Fitting results

We can check that the results of the parameter inference process are at least reasonable by looking at the Markov chain Monte Carlo chain plots for the model parameters:

>> tl = axisLabels "iter" "a" $ chainPlot (par<#>a)
>> tr = axisLabels "iter" "b" $ chainPlot (par<#>b)
>> bl = axisLabels "iter" "c" $ chainPlot (par<#>c)
>> br = axisLabels "iter" "Variance" $ chainPlot (par<#>variance)

?> PlotColumn [] [PlotRow [] [tl, tr], PlotRow [] [bl, br]]

All of these chains look well-mixed (no systematic patterns, exploring a good range of parameter values), so we can have some confidence that the regression results are reasonable.


## Interpretation and diagnostics

We can now look at the model regression results.

#### Parameter distributions

First, let's look at the distributions of the three regression parameters, $a$, $b$ and $c$:

>> adist = axisLabels "a" "density" $ distPlot $ with par a
>> bdist = axisLabels "b" "density" $ distPlot $ with par b
>> cdist = axisLabels "c" "density" $ distPlot $ with par c

?> PlotRow [] [adist, bdist, cdist]

Just from eyeballing these plots, it seems that all of the model parameters are significantly different from zero, but we can ask directly what the probability of, for example, finding a value of $a$ less than 50 mm is:

?> with par $ a < 50

We can ask similar questions about the other parameters.

#### Model + data plots

A more direct view of the relationship between the model and the data can be seen by plotting the data points we saw above on the same axis as a number of realisations of our model &mdash; it's important always to remember that what you get back from a call to `estimate` is a joint posterior distribution over the model parameters, which means that it's possible to sample from this distribution to generate realisations of our model.  We can then plot the "deterministic" part of the model for a number of realisations to get an idea of the range of behaviour that our model can represent.  In this plot, we show the data (blue points) and realisations of the model (lines) to get an idea of how well the distribution represented by our model fit covers the data:

?> axisLabels "Age (yr)" "Jaw size (mm)" $ over
?>    [plines $ with par $ with deer (x, a - b * exp (-c * x)),
?>     scatterPlot $ map (\{..} -> (x, y)) deer]

#### Residuals

Finally, we can get some idea of whether there are systematic effects in the data not represented by our model by looking for patterns in residuals.  Here we plot residuals of jaw size from the model compared to measured values against age:

?> axisLabels "Age (yr)" "Jaw size residuals (mm)" $
?>   ppoints $ with par $ with deer (x, y - (a - b * exp (-c * x)))

It's clear that there is no systematic variation in the data unexplained by our model.


## Conclusion

There is more that could be said about this example.  In particular, our three-parameter model can be simplified into a two-parameter model.  We'd then like some means to decide which of these models is "better", balancing fit to the data against model simplicity.  We'll cover this kind of question in a later article.  For the moment, this article should give you some idea of how easy it is to do nonlinear regression in BayesHive.  As mentioned above, all of the Baysig code shown here can be produced automatically by the BayesHive nonlinear regression model builder meaning that you don't need to write any code at all!
