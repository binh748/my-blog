---
title: "Forecasting monthly sales using SARIMA model"
categories:
  - Blog
tags:
  - SARIMA
  - ARIMA
  - Python
  - time series
  - statsmodels
excerpt:
---
## Table of Contents
1. [Introduction](#introduction)
2. [SARIMA Models Explained](#sarima-models-explained)
3. [Project Walkthrough](#project-walkthrough)
4. [Conclusion](#conclusion)

## Introduction
During the Metis data science bootcamp, we only spent a day on time series modeling, but it was an interesting subject for me since I had previously worked at D. E. Shaw where time series modeling of financial instruments was an integral part of D. E. Shaw’s quantitative forecasts. I worked on the trading floor of D. E. Shaw and remember finding a thick textbook on time series modeling that someone had left near the pantry (pantry is what we call the snacks and drinks area at D. E. Shaw). I flipped through the book, and I was amazed by how there was so much information on just time series modeling; after my Metis lecture on time series modeling, I understood why.

After bootcamp, I committed myself to continue learning and keeping my skills sharp, for example, by continuing to do projects. For my first post-bootcamp project, I wanted to revisit time series modeling, learn it more in-depth, and build my own time series model. To start, I found a small Kaggle dataset of superstore sales. Before diving in, I reviewed Metis’s instructional time series modeling notebook and this extremely helpful [video tutorial](https://youtu.be/3Gw1E_BJU58) from a former Metis instructor.

## SARIMA Models Explained
I used a SARIMA model for this project, which is an ARIMA model with a seasonal component. But what’s an ARIMA model, and what do I mean by seasonal component? I’ll explain, starting with what an ARIMA model is.

ARIMA is an acronym that stands for Autoregressive Integrated Moving Average. Autoregressive refers to autoregressive models, integrated refers to the orders of differencing, and moving average refers to moving-average models. You may find this strange, but I think of an ARIMA model as cerberus, the three-headed dog that guards the gates of the underworld for Hades: the first head is the autoregressive model (AR), the second head is the orders of differencing (I), and the third head is the moving-average model (MA). These three components are combined into the cute version of Cerberus you see below.

![arima-cerberus](https://user-images.githubusercontent.com/62628676/103049248-3cd90f80-455f-11eb-81f9-018e3d2fbaf3.png)

Let’s go through each ARIMA component, although by no means are my explanations exhaustive. You can take a whole graduate course on time series modeling!

### Autoregressive Models
An autoregressive model is premised upon a target variable that is serially correlated. In other words, you can use previous target observations to predict the current target observation because they’re highly correlated with each other. For example, weekly box office gross could be modeled using an autoregressive model as we would expect this week’s box office gross to be dependent on the prior week’s or weeks’.

The autoregressive model models the target as a linear combination of previous target observations as features plus an intercept and error term. An AR-1 model is an autoregressive model with one lagged target term (lag means the feature is from a previous time period), an AR-2 model has two lagged target terms, and so on.

Autoregressive models are linear, so we can use sum of squared errors (OLS) as our loss function like we do with linear regression and use gradient descent to find the model parameters (i.e. the coefficients, which are denoted as phi, and intercept) that minimize the loss function.

$$ X_t = c + \varphi_{t-1}X_{t-1} + \varphi_{t-2}X_{t-2} + \varepsilon_t $$

<span style="font-size: .8em; font-style: italic;">Equation for an AR-2 model. Note that we use phi for the coefficients in AR models.</span>

### Integrated
Integrated refers to the orders of differencing used in an ARIMA model. One order of differencing means we calculate the difference between the adjacent target values, two orders of differencing means we calculate the difference between the one order differencings, and so on. But why and when do we need to use differencing?

The ARIMA model, specifically the MA component that refers to the moving-average model (more on that soon), assumes a stationary time series, which means that the time series has constant mean, constant variance, and constant covariance throughout time. If the time series is not stationary (there are various ways we can check if a time series is stationary; more on that soon as well), then we continue differencing until we achieve a stationary time series.

One thing to note: if we use differencing, that means our ARIMA model will be predicting the differences rather than the actual target values, but packages like statsmodel in Python will know to use those differences to calculate the predicted target values.

### Moving-Average Models
For me and probably most people, the moving-average model is the most confusing component of ARIMA models, so let’s walk through it slowly.

The assumption behind moving-average models is that certain time series fluctuate randomly around a mean; in other words, these types of time series are mean-reverting. For example, let’s say we have a time series of monthly manga sales for a particular manga where there’s a constant mean of 10,000 manga sold per month. Although the mean is constant throughout time, the monthly sales deviate above and below the mean. In one month, sales might be high because there was a popular manga convention where lots of copies of the manga were sold; in another month, sales were depressed because there was a delay in the release of the newest manga volume. Despite these up-and-down fluctuations, the mean of this time series is consistently 10,000 manga sold per month.

What the moving-average model captures is the randomness (i.e. the stochastic nature) of the time series by modeling the target as a linear combination of the mean, the current error term, and previous error terms. Similar to how AR models are abbreviated, an MA-1 model has one lagged error term, an MA-2 model has two lagged error terms, and so on. By using lagged error terms, our predictions incorporate the information from past errors.

$$ X_t = \mu + \varepsilon_t + \theta_{t-1}\varepsilon_{t-1} + \theta_{t-2}\varepsilon_{t-2} $$

<span style="font-size: .8em; font-style: italic;">Equation for an MA-2 model. Note that we use theta for the coefficients in MA models.</span>

Let’s go back to our manga example with a mean of 10,000 monthly sales. If manga sales were 20,000 for a month because of the popular manga convention, but we had predicted that it would be 10,000, then our error would be 10,000. For next month’s prediction, we know that we underestimated last month’s monthly sales, so we’ll incorporate our previous error (let’s assume the previous error’s coefficient, theta, is 0.5) to adjust for that underestimation, so our next month’s prediction is 15,000 compared with next month’s true monthly sales of 16,000. Pretty cool, right?—as we incorporated information from our previous error, our prediction got closer to the true value.

$$ M_t = \mu + \varepsilon_t + 0.5\varepsilon_{t-1} $$

<span style="font-size: .8em; font-style: italic; display: block;">MA-1 model for monthly manga sales.</span>

| $t$ | $\hat{M_t}$ | $\varepsilon_t$ |  $M_t$ |
|:---:|:-----------:|:---------------:|:------:|
|  1  |    10,000   |      10,000     | 20,000 |
|  2  |    15,000   |      1,000      | 16,000 |

<span style="font-size: .8em; font-style: italic; display: block;">\\(\hat{M_t}\\) is predicted manga sales, \\(\varepsilon_t\\) is the error, and \\(M_t\\) is true manga sales.</span>

The manga example works because 1) I used the mean to initialize my first prediction, which allowed me to calculate my first error and 2) I already had a fitted MA model to work with. But how do I fit a MA model? This is where things get more complicated. Looking at the equation for a MA model, you may think it’s a linear model since it uses a linear combination of terms just like the AR model, but it’s not! The reason it’s not linear is because the lagged error terms are not observed, which means we can’t use OLS and gradient descent to fit a MA model.

What do I mean by the lagged error terms are not observed? Let’s assume a MA-1 model. To get the first prediction, I need the previous error term, but I don’t have it since I’m on the first prediction; to make the second prediction, I need the first prediction’s error, but I could not make the first prediction to begin with. That’s what I mean by the lagged error terms are not observed. To fit a MA model, we have to use a non-linear fitting process to estimate the error terms. I must admit I’m not sure how this non-linear fitting process is done, but the important takeaway here is that good ol’ OLS and gradient descent cannot be used to fit an MA model. One other thing I must admit is that even with a fitted MA model, I’m not sure how the predictions are initialized since you still need initial predictions to calculate the errors—maybe they’re initialized with the mean like in my manga example? The problem of calculating errors also applies to forecasting with a MA model: if you’re forecasting into the future, then you don’t know what the errors are for those forecasts since you don’t know what the true values are. If you’re able to fill the holes in my understanding of MA models, please leave a response in the comments section. Greatly appreciate it!

Awesome! We’ve gone through all the components of an ARIMA model. Let’s summarize and put it all together before we move on to understanding how a seasonal ARIMA model works.

An ARIMA model assumes that a time series can be modeled using lagged target observations (AR model) and lagged error terms (MA model). In other words, a time series can have a serial correlation component to them, which is shown by their trends (up/down/flat), and a stochastic/random component to them. Finally, because ARIMA models, specifically the MA portion of ARIMA models, assume a stationary time series, we must use \\(d\\)-orders of differencing until we achieve stationarity. An ARIMA model can be summarized using a shorthand: \\((p, d, q)\\) where \\(p\\) is the number of AR terms, \\(d\\) is the orders of differencing, and \\(q\\) is the number of MA terms. Thus, an (1, 1, 2) ARIMA model has 1 AR term, 1 order of differencing, and 2 MA terms.

We’ve now arrived at the last component that a time series can be made up of, and that’s seasonality.

### Seasonality
Seasonality refers to predictable variations in a time series that occur at intervals less than a year. Monthly average temperature in the northeastern United States, for example, would have seasonality as the hottest monthly temperatures are in the summer months and the coldest monthly temperatures are in the winter months, and this seasonal trend repeats annually. Monthly retail sales would also have seasonality since retail sales are typically highest in the holiday months as people go holiday shopping.

In order to capture the seasonality component of a time series, we use a SARIMA model, which has seasonal ARIMA components \\((P, D, Q)m\\) in addition to normal ARIMA components \\((p, d, q)\\). You may have noticed that \\(m\\) is the only new letter: it is the number of time periods in a season. For example, if we are forecasting monthly sales, then \\(m=12\\) as there are 12 months in a year where one year is a complete season. \\(P, D, Q\\) are the same as \\(p, d, q\\) in an ARIMA model except the \\(P, D, Q\\) terms are using seasonal lags based on what \\(m\\) is. For example, if \\(P=2\\), then that means we have 2 SAR (seasonal autoregressive) terms where the first term refers to the target observation 12 months ago and the second term refers to the target observation 24 months ago. This makes sense because if I want to predict December monthly sales using a seasonal AR model, I would want to use December monthly sales from a year ago and potentially December monthly sales from two years ago. The same logic of looking at prior seasons applies to the orders of seasonal differencing and the SMA (seasonal moving-average) terms. All the components of a SARIMA model can be summarized using this shorthand: \\((p, d, q) (P, D, Q, m)\\).

Brilliant! We now have a solid foundation on how SARIMA models work.

## Project Walkthrough
The data for this project was pulled from [Kaggle](https://www.kaggle.com/rohitsahoo/sales-forecasting). The data consists of four years of superstore sales from the start of 2015 to the end of 2018. After doing EDA and light data cleaning (the data was mostly clean to start with), I grouped the sales, which were recorded at the individual order level, into monthly sales because forecasting monthly sales (as opposed to daily or weekly sales) seemed to me to be the most realistic business problem.

In the figure below of monthly sales, there is a clear seasonal pattern: the highest monthly sales occur annually in the holiday months of November and December. This means that once I start modeling, I’ll have to use a SARIMA model to capture the seasonality.

![monthly-superstore-sales](https://user-images.githubusercontent.com/62628676/103041681-1ad29380-4545-11eb-847a-7c66f38b2034.png)
<span style="font-size: .8em; font-style: italic; display: block;">Monthly sales are highest in November and December, indicating seasonality.</span>


To check if the monthly sales are stationary, I used three different checks. First, I applied a visual check where I plotted the original time series against the 12-month rolling mean and 12-month rolling standard deviation to see if the rolling mean and standard deviation stay constant. In the figure below, the rolling standard deviation appears constant, but the rolling mean does have an upward trend starting in mid-2017. The next check was a standard deviation check. The differenced time series with the lowest standard deviation is usually the most stationary. I calculated the standard deviation for zero up to three orders of differencing: the standard deviation with zero orders of differencing was the lowest. Finally, I ran the Dickey-Fuller test, which is the most statistically rigorous method to determine stationarity. The null hypothesis of the Dickey-Fuller test is that the time series is not stationary. The p-value for my Dickey-Fuller test was 0.000278, well below the standard 0.05 significance level; thus, I reject the null hypothesis in favor of the alternative that the time series is stationary.

![rolling-mean-rolling-std-dev](https://user-images.githubusercontent.com/62628676/103041715-32aa1780-4545-11eb-90a7-76646a1503ef.png)
<span style="font-size: .8em; font-style: italic; display: block;">Per this visual check, it does not look like the time series is stationary as the rolling standard deviation is constant, but the rolling mean is not.</span>

Not all the stationarity checks agreed with each other (the visual check showed that the time series was not stationary due to the upward trend in the rolling mean), but the last two checks did concur that the time series was stationary enough. Let’s run with the conclusion for now that the monthly superstore stores, without any orders of differencing, is stationary.

One point of clarification is that the checks I did were only to verify that the time series itself is stationary; I did not check if the time series’ seasonal differencing is stationary. I will later check for this when I start fitting my model. Duke University has a great explanation on seasonal differencing that I recommend you to read [here](https://people.duke.edu/~rnau/411sdif.htm).

Before I explain how I trained my models, there’s one point I should mention. Unlike training a regression or classification model, you cannot use sklearn’s train_test_split() function because it randomizes the data before making the splits. For time series models, you can’t randomize the data because you want to use the beginning portion of your time series as your training set to predict the remaining portion as the validation/test set. This makes sense since a time series model allows you to use past data to predict future data, and that doesn’t work if you’re randomizing your data such that past and future data get mixed together.

sklearn does provide a class to properly do time series splitting called TimeSeriesSplit(), but TimeSeriesSplit() uses k-fold cross-validation where I would progressively train on larger portions of data and do validation on a different set each time. Because my time series consists of only four years of data, I didn’t have enough data to do cross-validation. Instead, I used simple validation and manually split my time series data in a 50/25/25 ratio into a training set (24 months), validation set (12 months), and test set (12 months). However, to train a model on only 24 months of data is not ideal when the time series has a seasonal component: 24 months of data only contains two seasons, which means my model won’t be able to learn the seasonal pattern that well. It was at this point that I realized I should have used a larger time series dataset, but I decided to work with what I had and see what my results look like.

We’re now at the point where I train my models. Using statsmodel, I created autocorrelation and partial autocorrelation plots (aka ACF and PACF plots where ACF stands for autocorrelation function and PACF stands for partial autocorrelation functions). Knowing how to interpret these plots is absolutely necessary for time series modeling, so let’s do a quick 101. An autocorrelation plot shows the total correlation between the time series and lagged versions of itself for a specified number of lags. For example, an autocorrelation at lag 2 of 0.47 means that if you take the monthly sales and correlate it with monthly sales from two months ago (i.e. if you created a column of monthly sales and correlated it with another column where for each monthly sale, it’s the monthly sales from two months ago), you get a Pearson correlation coefficient of 0.47. The partial autocorrelation plot is the same as the autocorrelation plot except that you remove the indirect correlation between the time series and the lagged version of itself. By indirect correlation, I mean the correlation that is the result of a lagged version being correlated with the next lagged version and so on until you reach the time series. Thus, going back to the example where you have an autocorrelation at lag 2 of 0.47, the partial autocorrelation at lag 2 may only be 0.24 once you remove the indirect correlation that is a byproduct of lag 2 being correlated with lag 1 and lag 1 being correlated with the time series.

![acf-pacf-plots](https://user-images.githubusercontent.com/62628676/103041926-db587700-4545-11eb-9e96-a2b3ca0fa1e2.png)

Using the ACF and PACF plots, I referenced [Duke University’s rules](https://people.duke.edu/~rnau/arimrule.htm) on how to fit an SARIMA model. The rules tell you prescriptively how to choose the terms for your SARIMA model based on your ACF and PACF plots. To evaluate my models during the fitting process, I used [AIC (Akaike information criterion)](https://en.wikipedia.org/wiki/Akaike_information_criterion) as my metric; the lower the AIC, the better. I applied the Duke University’s rules until I got the AIC to be as low as it could be. Below you can see the comments in my code on how I applied the rules.

![cell-with-duke-rules](https://user-images.githubusercontent.com/62628676/103042169-ba445600-4546-11eb-8678-59fa9d799f99.png)

Using my training set, I trained two different models that had similarly low AICs: a SARIMA model with parameters (0, 0, 0) (0, 1, 1, 12) and a second model with parameters (0, 1, 1) (0, 1, 1, 12). As a refresher, the SARIMA model shorthand is \\((p, d, q) (P, D, Q, m)\\). Both models have one order of seasonal differencing and one SMA term, but the second model also has an  order of (non-seasonal) differencing and a MA term. We did see that when we performed our stationarity checks that we wouldn’t need to use an order of differencing, so which model do you think outperforms the other?

Before validation, I ran statistical tests on both models to check that the [in-sample](https://stats.stackexchange.com/questions/260899/what-is-difference-between-in-sample-and-out-of-sample-forecasts) residuals based on one-step-ahead (aka point) predictions  are homoscedastic, independent (i.e. no serial correlation), and normally distributed, which are the same assumptions for the residuals in linear regression. I applied the Jarque-Bera normality test, the Ljung-Box serial correlation test, and the heteroskedasticity test, and both models passed all three tests.

I validated the models on the validation set I created earlier to see which one performs better. To do the validation, I started my forecast at the first data point in the validation set and ended on the last. A note on terminology: although there is some debate on the difference in meaning between predictions and forecasts, in time series modeling, the consensus generally is that model outputs for in-sample data are called predictions whereas model outputs for out-of-sample data are called forecasts (if you’re interested in the debate, see [here](https://stats.stackexchange.com/questions/65287/difference-between-forecast-and-prediction)). Using root mean squared error (RMSE) as my forecast error metric, the (0, 0, 0) (0, 1, 1, 12) SARIMA model outperformed the (0, 1, 1) (0, 1, 1, 12) model with a validation RMSE of $17k compared with $31k. Now we know that the additional order of differencing did not help, which had been indicated to us earlier by our stationarity checks.

Last, I tested the (0, 0, 0) (0, 1, 1, 12) SARIMA model on the test set and got a test RMSE of $15k. Not bad considering the test RMSE is 0.57 standard deviations of the test set’s monthly sales, but not amazing either.

![test-sarima-model](https://user-images.githubusercontent.com/62628676/103045150-5d01d200-4551-11eb-8408-a3d171b7c2bc.png)

## Conclusion
This was my first post-bootcamp project, and I’m really proud I did it from beginning to end. In the process, I spent hours learning the fundamentals of SARIMA time series modeling and trained, validated, and tested a SARIMA model of my own. I’m sure I’ll do more time series modeling as I advance in my data science career.

All code on [GitHub](https://github.com/binh748/superstore-sales).

*Closing note: I tried my best to explain key time series modeling concepts. As I noted earlier, there are holes in my understanding of moving-average models. If you’re able to fill in the details there or if I made a mistake or omission elsewhere, please write in the comments section below. Really grateful for your readership.*
