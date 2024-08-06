
# CAPM Analysis

## Introduction

In this assignment, you will explore the foundational concepts of the Capital Asset Pricing Model (CAPM) using historical data for AMD and the S&P 500 index. This exercise is designed to provide a hands-on approach to understanding how these models are used in financial analysis to assess investment risks and returns.

## Background

The CAPM provides a framework to understand the relationship between systematic risk and expected return, especially for stocks. This model is critical for determining the theoretically appropriate required rate of return of an asset, assisting in decisions about adding assets to a diversified portfolio.

## Objectives

1. **Load and Prepare Data:** Import and prepare historical price data for AMD and the S&P 500 to ensure it is ready for detailed analysis.
2. **CAPM Implementation:** Focus will be placed on applying the CAPM to examine the relationship between AMD's stock performance and the overall market as represented by the S&P 500.
3. **Beta Estimation and Analysis:** Calculate the beta of AMD, which measures its volatility relative to the market, providing insights into its systematic risk.
4. **Results Interpretation:** Analyze the outcomes of the CAPM application, discussing the implications of AMD's beta in terms of investment risk and potential returns.

## Instructions

### Step 1: Data Loading

- We are using the `quantmod` package to directly load financial data from Yahoo Finance without the need to manually download and read from a CSV file.
- `quantmod` stands for "Quantitative Financial Modelling Framework". It was developed to aid the quantitative trader in the development, testing, and deployment of statistically based trading models.
- Make sure to install the `quantmod` package by running `install.packages("quantmod")` in the R console before proceeding.

```r
# Set start and end dates
start_date <- as.Date("2019-05-20")
end_date <- as.Date("2024-05-20")

# Load data for AMD, S&P 500, and the 1-month T-Bill (DTB4WK)
amd_data <- getSymbols("AMD", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
gspc_data <- getSymbols("^GSPC", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
rf_data <- getSymbols("DTB4WK", src = "FRED", from = start_date, to = end_date, auto.assign = FALSE)

# Convert Adjusted Closing Prices and DTB4WK to data frames
amd_df <- data.frame(Date = index(amd_data), AMD = as.numeric(Cl(amd_data)))
gspc_df <- data.frame(Date = index(gspc_data), GSPC = as.numeric(Cl(gspc_data)))
rf_df <- data.frame(Date = index(rf_data), RF = as.numeric(rf_data[,1]))  # Accessing the first column of rf_data

# Merge the AMD, GSPC, and RF data frames on the Date column
df <- merge(amd_df, gspc_df, by = "Date")
df <- merge(df, rf_df, by = "Date")
```

#### Data Processing 
```r
colSums(is.na(df))
# Fill N/A RF data
df <- df %>%
  fill(RF, .direction = "down") 
```

### Step 2: CAPM Analysis

The Capital Asset Pricing Model (CAPM) is a financial model that describes the relationship between systematic risk and expected return for assets, particularly stocks. It is widely used to determine a theoretically appropriate required rate of return of an asset, to make decisions about adding assets to a well-diversified portfolio.

#### The CAPM Formula
The formula for CAPM is given by:

$$
E(R_i) = R_f + \beta_i (E(R_m) - R_f)
$$

Where:

- $E(R_i)$ is the expected return on the capital asset,
- $R_f$ is the risk-free rate,
- $\beta_i$ is the beta of the security, which represents the systematic risk of the security,
- $E(R_m)$ is the expected return of the market.



#### CAPM Model Daily Estimation

- **Calculate Returns**: First, we calculate the daily returns for AMD and the S&P 500 from their adjusted closing prices. This should be done by dividing the difference in prices between two consecutive days by the price at the beginning of the period.
  
$$
\text{Daily Return} = \frac{\text{Today's Price} - \text{Previous Trading Day's Price}}{\text{Previous Trading Day's Price}}
$$

```r
# Daily Return, Daily Risk-free Rate & Excess Return Calculation
for (i in 1:nrow(df)) {
 if (df$Date[i] == start_date) {
 # Initialisation of variables
 daily_return_AMD <- NA
 daily_return_GSPC <- NA

 } else {
 # Daily return calculations
 daily_return_AMD <- (df$AMD[i] - df$AMD[i-1])/(df$AMD[i-1])
 daily_return_GSPC <- (df$GSPC[i] - df$GSPC[i-1])/(df$GSPC[i-1])
 }

 # Inputting returns onto data frame (df)
 df$daily_return_AMD[i] <- daily_return_AMD
 df$daily_return_GSPC[i] <- daily_return_GSPC

}
```

- **Calculate Risk-Free Rate**: Calculate the daily risk-free rate by conversion of annual risk-free Rate. This conversion accounts for the compounding effect over the days of the year and is calculated using the formula:
  
$$
\text{Daily Risk-Free Rate} = \left(1 + \frac{\text{Annual Rate}}{100}\right)^{\frac{1}{360}} - 1
$$

```r
# Daily Return, Daily Risk-free Rate & Excess Return Calculation
for (i in 1:nrow(df)) {
 if (df$Date[i] == start_date) {
 # Application of Daily Risk-free Rate Formula
 daily_return_RF <- (((1 + df$RF[1]/100)^(1/360)) - 1)

 } else {
 # Application of Daily Risk-free Rate Formula
 daily_return_RF <- (((1 + df$RF[i]/100)^(1/360)) - 1)
 }

 # Inputting returns onto data frame (df)
 df$daily_return_RF[i] <- daily_return_RF

}

```


- **Calculate Excess Returns**: Compute the excess returns for AMD and the S&P 500 by subtracting the daily risk-free rate from their respective returns.

```r
# Daily Return, Daily Risk-free Rate & Excess Return Calculation
for (i in 1:nrow(df)) {
 if (df$Date[i] == start_date) {
 # Initialisation of variables
 daily_excess_return_AMD <- NA
 daily_excess_return_GSPC <- NA

 } else {
 # Excess return calculations
 daily_excess_return_AMD <- df$daily_return_AMD[i] - df$daily_return_RF[i]
 daily_excess_return_GSPC <- df$daily_return_GSPC[i] - df$daily_return_RF[i]
 }

 # Inputting returns onto data frame (df)
 df$daily_excess_return_AMD[i] <- daily_excess_return_AMD
 df$daily_excess_return_GSPC[i] <- daily_excess_return_GSPC

}
```


- **Perform Regression Analysis**: Using linear regression, we estimate the beta (\(\beta\)) of AMD relative to the S&P 500. Here, the dependent variable is the excess return of AMD, and the independent variable is the excess return of the S&P 500. Beta measures the sensitivity of the stock's returns to fluctuations in the market.

```r
# Initialising columns
excess_GSPC <- df$daily_excess_return_GSPC
excess_AMD <- df$daily_excess_return_AMD
# Applying regression analsis to AMD & S&P500
mod1 <- lm(excess_AMD ~ excess_GSPC, data = df)
summary(mod1) 

```


#### Interpretation

What is your \(\beta\)? Is AMD more volatile or less volatile than the market?

**Answer:**
Beta measures a particular stock’s volatility relative to the S&P 500, determined by the slope of the regression line.
A beta value of 1.56999 means that a 1% increase in excess share returns in the market, results in a roughly 1.57% increase in AMD excess
returns, making AMD’s returns more sensitive to market changes. Thus, changes in the market would cause even larger changes in AMD’s returns.
As a result, AMD would be considered more volatile than the market.


#### Plotting the CAPM Line
Plot the scatter plot of AMD vs. S&P 500 excess returns and add the CAPM regression line.

```r
# Plotting Data Plot & Regression Line
plot(excess_GSPC, excess_AMD, pch=1, cex=1, col = "blue", xlab = "S&P 500", ylab = "AMD", main = "AMD vs S&P 500
(Excess Returns)")
abline(mod1, lwd=3, col='red')

```

### Step 3: Predictions Interval
Suppose the current risk-free rate is 5.0%, and the annual expected return for the S&P 500 is 13.3%. Determine a 90% prediction interval for AMD's annual expected return.



**Answer:**

```r
# Beta Values
beta_0 <- summary(mod1)$coefficients[1, 1] # Intercept (beta_0)
beta_1 <- summary(mod1)$coefficients[2, 1] # Slope (beta_1)
# Other values
rf_rate <- 0.05
expected_market_return <- 0.133
# AMD Annual Expected Return (USING CAPM FORULA)
AMD_expected_return_annual <- (rf_rate) + (beta_1)*(expected_market_return - rf_rate)
# Conversion from annual to daily
AMD_expected_return_daily <- AMD_expected_return_annual / 252
# Number of observations
n <- length(df$daily_excess_return_GSPC)
# Mean of AMD Excess Returns
mean_GSPC <- mean(df$daily_excess_return_GSPC, na.rm = TRUE)
# Standard error of the estimate
se <- sqrt(sum(residuals(mod1)^2) / (n-2))
# Sum of squares of the AMD excess return values
SSX <- sum(((df$daily_excess_return_GSPC - mean_GSPC)^2), na.rm = TRUE)
# Calculate the standard error of the forecast
sf_daily <- se*sqrt(1 + 1/n + (AMD_expected_return_daily - mean_GSPC)^2 / SSX)
# Conversion from daily to annual
sf_annual <- sqrt(252) * sf_daily
# 90% confidence level
alpha <- 0.10
# t value for 90% confidence level with n-2 degrees of freedom
t_value <- qt(1 - alpha/2, df = n - 2)
# Prediction interval
lower_bound <- AMD_expected_return_annual - (t_value * sf_annual)
upper_bound <- AMD_expected_return_annual + (t_value * sf_annual)
# Print results
print(lower_bound)

```
