
## Algorithmic Trading Strategy

## Introduction

In this assignment, you will develop an algorithmic trading strategy by incorporating financial metrics to evaluate its profitability. This exercise simulates a real-world scenario where you, as part of a financial technology team, need to present an improved version of a trading algorithm that not only executes trades but also calculates and reports on the financial performance of those trades.

## Background

Following a successful presentation to the Board of Directors, you have been tasked by the Trading Strategies Team to modify your trading algorithm. This modification should include tracking the costs and proceeds of trades to facilitate a deeper evaluation of the algorithm’s profitability, including calculating the Return on Investment (ROI).

After meeting with the Trading Strategies Team, you were asked to include costs, proceeds, and return on investments metrics to assess the profitability of your trading algorithm.

## Objectives

1. **Load and Prepare Data:** Open and run the starter code to create a DataFrame with stock closing data.

2. **Implement Trading Algorithm:** Create a simple trading algorithm based on daily price changes.

3. **Customize Trading Period:** Choose your entry and exit dates.

4. **Report Financial Performance:** Analyze and report the total profit or loss (P/L) and the ROI of the trading strategy.

5. **Implement a Trading Strategy:** Implement a trading strategy and analyze the total updated P/L and ROI. 

6. **Discussion:** Summarise your finding.


## Instructions

### Step 1: Data Loading

Start by running the provided code cells in the "Data Loading" section to generate a DataFrame containing AMD stock closing data. This will serve as the basis for your trading decisions. First, create a data frame named `amd_df` with the given closing prices and corresponding dates. 

```r
# Load data from CSV file
amd_df <- read.csv("AMD.csv")
# Convert the date column to Date type and Adjusted Close as numeric
amd_df$date <- as.Date(amd_df$Date)
amd_df$close <- as.numeric(amd_df$Adj.Close)
amd_df <- amd_df[, c("date", "close")]
```

#### Plotting the Data
Plot the closing prices over time to visualize the price movement.
```r
plot(amd_df$date, amd_df$close,'l')
```

### Step 2: Trading Algorithm
Implement the trading algorithm as per the instructions. You should initialize necessary variables, and loop through the dataframe to execute trades based on the set conditions.

- Initialize Columns: Start by ensuring dataframe has columns 'trade_type', 'costs_proceeds' and 'accumulated_shares'.
- Change the algorithm by modifying the loop to include the cost and proceeds metrics for buys of 100 shares. Make sure that the algorithm checks the following conditions and executes the strategy for each one:
  - If the previous price = 0, set 'trade_type' to 'buy', and set the 'costs_proceeds' column to the current share price multiplied by a `share_size` value of 100. Make sure to take the negative value of the expression so that the cost reflects money leaving an account. Finally, make sure to add the bought shares to an `accumulated_shares` variable.
  - Otherwise, if the price of the current day is less than that of the previous day, set the 'trade_type' to 'buy'. Set the 'costs_proceeds' to the current share price multiplied by a `share_size` value of 100.
  - You will not modify the algorithm for instances where the current day’s price is greater than the previous day’s price or when it is equal to the previous day’s price.
  - If this is the last day of trading, set the 'trade_type' to 'sell'. In this case, also set the 'costs_proceeds' column to the total number in the `accumulated_shares` variable multiplied by the price of the last day.



```r
# Initialize columns for trade type, cost/proceeds, and accumulated shares in amd_df
amd_df$trade_type <- NA
amd_df$costs_proceeds <- NA  # Corrected column name
amd_df$accumulated_shares <- 0  # Initialize if needed for tracking

# Initialize variables for trading logic
previous_price <- 0
share_size <- 100
accumulated_shares <- 0

for (i in 1:nrow(amd_df)) {
 if (i == 1) {
 # When looping through the 1st row, let previous price = 0
 previous_price <- 0
 } else {
 # When looping through any other row, let previous price = prior closing price
 previous_price <- amd_df$close[i - 1]
 }


 # If start date has been reached, 100 shares are purchased
 if (amd_df$date[i] == start_date) {
 # Set trade type to 'buy'
 amd_df$trade_type[i] <- 'buy'
 # Record the cost of buying shares (negative value indicates cost)
 amd_df$costs_proceeds[i] <- -(amd_df$close[i] * share_size)
 # Update the total number of shares held
 accumulated_shares <- (accumulated_shares + share_size)

 # If end date has been reached, all shares must be sold to evaluate financial metrics
 } else if (amd_df$date[i] == end_date) {
 # Set trade type to 'sell'
 amd_df$trade_type[i] <- 'sell'
 # Record the proceeds from selling all accumulated shares (positive value indicates proceeds)
 amd_df$costs_proceeds[i] <- (amd_df$close[i] * accumulated_shares)
 # Reset the total number of shares held to 0
 accumulated_shares <- 0


 # If the current day's price is less than the previous day's price
 } else if (amd_df$close[i] < previous_price) {
 # Set trade type to 'buy'
 amd_df$trade_type[i] <- 'buy'
 # Record the cost of buying shares (negative value indicates cost)
 amd_df$costs_proceeds[i] <- -(amd_df$close[i] * share_size)
 # Update the total number of shares held
 accumulated_shares <- (accumulated_shares + share_size)
 }


 # In the event, shares have been purchased, investor capital is updated
 if (amd_df$trade_type[i] == 'buy' && !is.na(amd_df$trade_type[i])) {
 investor_capital <- investor_capital + (-amd_df$costs_proceeds[i])
 } else {
 investor_capital <- investor_capital
 }

 # In the event shares have been sold, revenue is updated
 if (amd_df$trade_type[i] == 'sell' && !is.na(amd_df$trade_type[i])) {
 revenue <- revenue + amd_df$costs_proceeds[i]
 } else {
 revenue <- revenue

 # Store the updated data onto the DataFrame
 amd_df$accumulated_shares[i] <- accumulated_shares
 }
}
```


### Step 3: Customize Trading Period
- Define a trading period you wanted in the past five years 
```r
# Define trading period
start_date <- as.Date('2020/01/02')
end_date <- as.Date('2021/12/02')
# Restrict amd_df to trading period
amd_df <- amd_df[(amd_df$date >= start_date) & (amd_df$date <= end_date),]
```


### Step 4: Run Your Algorithm and Analyze Results
After running your algorithm, check if the trades were executed as expected. Calculate the total profit or loss and ROI from the trades.

- Total Profit/Loss Calculation: Calculate the total profit or loss from your trades. This should be the sum of all entries in the 'costs_proceeds' column of your dataframe. This column records the financial impact of each trade, reflecting money spent on buys as negative values and money gained from sells as positive values.
- Invested Capital: Calculate the total capital invested. This is equal to the sum of the 'costs_proceeds' values for all 'buy' transactions. Since these entries are negative (representing money spent), you should take the negative sum of these values to reflect the total amount invested.
- ROI Formula: $$\text{ROI} = \left( \frac{\text{Total Profit or Loss}}{\text{Total Capital Invested}} \right) \times 100$$

```r
## PROFIT + ROI CALCULATIONS
# Calculations for profit & ROI
profit_1 <- (revenue - investor_capital)
investor_capital_1 <- investor_capital
ROI_1 <- (profit_1 / investor_capital) * 100
# Generate profit, investor_capital & ROI
print(profit_1)
```

### Step 5: Profit-Taking Strategy or Stop-Loss Mechanisum (Choose 1)
- Option 1: Implement a profit-taking strategy that you sell half of your holdings if the price has increased by a certain percentage (e.g., 20%) from the average purchase price.
- Option 2: Implement a stop-loss mechanism in the trading strategy that you sell half of your holdings if the stock falls by a certain percentage (e.g., 20%) from the average purchase price. You don't need to buy 100 stocks on the days that the stop-loss mechanism is triggered.


```r
## STEP 5 (USING STEP 3 & 2)
# Define trading period
start_date <- as.Date('2020/01/02')
end_date <- as.Date('2021/12/02')
# Restrict amd_df to trading period
amd_df <- amd_df[(amd_df$date >= start_date) & (amd_df$date <= end_date),]
# Initialize columns for trade type, cost/proceeds, average purchase price and accumulated shares in amd_df
amd_df$trade_type <- NA
amd_df$costs_proceeds <- NA # Corrected column name
amd_df$accumulated_shares <- 0 # Initialize if needed for tracking
amd_df$average_purchase_price <- NA
# Initialize variables for trading logic
share_size <- 100
accumulated_shares <- 0
average_purchase_price <- NA
qty_share_purchases <- 0
investor_capital <- 0
revenue <- 0
# Loop through each row in the DataFrame
for (i in 1:nrow(amd_df)) {

 if (amd_df$date[i] == start_date) {
 # When looping through the 1st row, let previous price = 0
 previous_price <- 0
 } else {
 # When looping through any other row, let previous price = prior closing price
 previous_price <- amd_df$close[i - 1]
 }


 # If start date has been reached, 100 shares are purchased
 if (amd_df$date[i] == start_date) {
 # Set trade type to 'buy'
 amd_df$trade_type[i] <- 'buy'
 # Record the cost of buying shares (negative value indicates cost)
 amd_df$costs_proceeds[i] <- -(amd_df$close[i] * share_size)
 # Update the total number of shares held
 accumulated_shares <- accumulated_shares + share_size


 # If end date has been reached, all shares must be sold to evaluate financial metrics
 } else if (amd_df$date[i] == end_date) {
 # Set trade type to 'sell'
 amd_df$trade_type[i] <- 'sell'
 # Record the proceeds from selling all accumulated shares (positive value indicates proceeds)
 amd_df$costs_proceeds[i] <- amd_df$close[i] * accumulated_shares
 # Reset the total number of shares held to 0
 accumulated_shares <- 0


 # Condition for strategy 1 (Profit-Taking Strategy)
 } else if (!is.na(average_purchase_price) && (amd_df$close[i] >= 1.2 * average_purchase_price)) {
 # Set trade type to 'sell'
 amd_df$trade_type[i] <- 'sell'
 # Update the total number of shares held
 accumulated_shares <- (1/2)*accumulated_shares
 # Record the proceeds from selling all accumulated shares (positive value indicates sales)
 amd_df$costs_proceeds[i] <- amd_df$close[i] * (accumulated_shares)

 # Condition for when to purchase 100 shares
 } else if (amd_df$close[i] < previous_price) {
 # Set trade type to 'buy'
 amd_df$trade_type[i] <- 'buy'
 # Record the cost of buying shares (negative value indicates cost)
 amd_df$costs_proceeds[i] <- -amd_df$close[i] * share_size
 # Update the total number of shares held
 accumulated_shares <- accumulated_shares + share_size
 }


 # In the event shares have been purchased, qty_share_purchases, investor_capital & average_purchase_price is
updated
 if (amd_df$trade_type[i] == 'buy' && !is.na(amd_df$trade_type[i])) {
 qty_share_purchases <- qty_share_purchases + share_size
 # sum of cost_proceeds column in data frame & NA values are disregarded divided by total share purchases
 average_purchase_price <- ((-sum(amd_df$costs_proceeds[amd_df$trade_type == 'buy'], na.rm = TRUE)) / qty_shar
e_purchases)
 investor_capital <- investor_capital + (-amd_df$costs_proceeds[i])
 } else {
 average_purchase_price <- average_purchase_price
 investor_capital <- investor_capital
 }


 # In the event shares have been sold, revenue is updated
 if (amd_df$trade_type[i] == 'sell' && !is.na(amd_df$trade_type[i])) {
 revenue <- revenue + amd_df$costs_proceeds[i]
 } else {
 revenue <- revenue

 # Store the updated data onto the DataFrame
 amd_df$accumulated_shares[i] <- accumulated_shares
 }
}

```


### Step 6: Summarize Your Findings
- Did your P/L and ROI improve over your chosen period?
- Relate your results to a relevant market event and explain why these outcomes may have occurred.


```r
## PROFIT + ROI CALCULATIONS
# Calculations for profit & ROI
profit_2 <- (revenue - investor_capital)
investor_capital_2 <- investor_capital
ROI_2 <- (profit_2 / investor_capital) * 100
# Generate profit, investor_capital & ROI
print(profit_2)
## Profit & ROI Comparison
# Change in profit
Change_profit <- profit_2 - profit_1
print(Change_profit)

```

As seen in the above plot, the share price of AMD surges to $140-160 near the end of the trading period. Since strategy 1 involves selling at the
end of the trading period, it was able to make significant profits and retain a considerably high ROI. However, for strategy 2, almost all shares were
sold by July 2020 (when the share price was fluctuating between $50-60) and hence, the strategy was unable to effectively capitalise on the
market boom.
This substantial growth in share price (during the end of the trading period) is attributable to fairly impressive financial figures released in their Q3
2021 report. Notably, AMD was holding 24.6% of CPU market share in Q3 2021, which was just short of its highest share dating back to Q4 2006.
This illustrates to investors, that this company has immense potential, boosting confidence which therefore results in a greater share price.
During 2021, there was also significant global demand for electronics, with global semiconductor sales reaching an all-time high in 2021 of USD
$553bn, a 26% increase from 2020. This is due to an increase in prevalence of households working from home and students engaging in remote
learning, which has risen demand for computers, as well as high demand for GPUs by crypto miners, due to sky-high cryptocurrency prices.
Accordingly, these events were beneficial towards AMD’s sales revenue which grew 54% (YoY), improving investor sentiment and demand for
stock.




