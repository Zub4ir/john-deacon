![img](prescient-logo-reversed-new.png)

# Welcome!

Thank you for deciding to honour us with your participation in the the Prescient Coding Challenge. We trust you've brought your thinking caps and are ready to tackle an interesting problem. Most of all, we hope this stimulates you mentally and that you have a lot of fun. We think the grand prize is worth it.

## The Mission

You are given a set of stocks and their monthly returns (return per stock per month) spanning from January 2010 up to last month. We challenge you to construct a portfolio that yields a higher [total return](https://www.investopedia.com/terms/t/totalreturn.asp) than an equally weighted index (the [benchmark index](https://www.investopedia.com/terms/b/benchmark.asp)). In other words, we want you to programmatically come up with a weighted portfolio index of stock returns each month that does better than if all stock returns in this index were just combined in equal weights through time. We will measure the outperformance on the test data only. 

## The Detail

You are given a training data set of dimensions $n_{train} \times (p + 1)$ and a test data set of dimensions and $n_{test} \times (p + 1)$ where $p$ is the number of stocks and $n$ is the length of the returns series for each stock. The monthly returns are in fractional format. 

The total return, $TR()$, on an equally weighted index for a particular month, $t$, is calculated as

$$TR(t) = 100 \times \Biggr[ \prod_{i=1}^{t\leq(n_{train} + n_{test})}  \biggr[ 1 + \Big( \frac{1}{p} \sum_{k=1}^{p} r_{ik} \Big) \biggr] \Biggr]$$

The challenge is to calculate a set of portfolio weights $w_{tk}$ for each month $t \in \{1,2,3..., n_{train} + n_{test}\}$ and stock $k \in \{1,2,3,...p\}$ such that

$$TR^{w}(t) = 100 \times \Biggr[ \prod_{i=1}^{t\leq(n_{train} + n_{test})}  \biggr[ 1 + \Big( \sum_{k=1}^{p} w_{ik}r_{ik} \Big) \biggr] \Biggr]$$

and

$$TR^{w}(n_{train} + n_{test}) \gt TR(n_{train} + n_{test})$$

where $\boldsymbol{w}$ is the matrix of weights of $w_{tk}$ and 

$$0 \leq w_{tk} \le 0.1 \: \forall \:t,k$$
$$ \sum_{k=1}^{p} w_{tk} = 1 \: \forall \:t$$

That is, the latest value in the portfolio's total return series is strictly greater than the corresponding value in the benchmark index's total return series.

## The Rules

1. As stated in the constraints above, no stock may have a weight higher than 10% (it can have a zero weight) at any point in time. Also, all weights must sum to 1 for any given month.
2. You are welcome and encouraged to use any external resources at your disposal (chatGPT, Google, stackoverflow, etc.).
3. You can use any methodology, machine learning, optimisation or algorithm of your choice. Make sure you have comments that explain what you are doing and *why*. The better we understand what you are attempting, the higher your chances of success.
4. You can use any IDE of your choice (VScode, spyder, RStudio, etc). It is the solution template that will matter at the end of the day.
5. We will evaluate your submission based on the following criteria:
   - Your methodology's performance against the equally weighted benchmark index.
   - How well you are able to articulate and explain your own answer. (Hint: Don't just throw all sorts of technical jargon onto a page. You don't have a lot of time. Be concise, show us how much of your answer you understand well.)
   - Generative AI is useful, but we will bias our evaluation towards more original answers. If you do use AI, use it to help you work smart. Don't try to have it do all the work for you.
6. We will also be keeping an eye out for [look-ahead bias](https://www.investopedia.com/terms/l/lookaheadbias.asp), hence the separation between test and training sets.

## The Data

You will find the data split into two `.csv` files, `returns_train.csv` and `returns_test.csv`:

- The data sets are not necessarily ordered by month end.
- The training set is the first part of the data, and the test set the second.
- There is a return for each stock at each month end.

## The Input

You are provided with a solution template to submit your answer in. If you are using R, work in `solution_skeleton.R`. For python, use `solution_skeleton.py`. The solution template contains three functions:

1. `equalise_weights()`: Generates a dataframe of equal weights for the index. 
2. `generate_portfolio()`: The skeleton or sample code to generate the portfolio weights.
3. `plot_total_return()`: Plot the total return series of both the index and portfolio.

The recomendation is to modify the weight generation section in the `generate_portfolio()` function (see below)

for python

```python
# <<--------------------- YOUR CODE GOES BELOW THIS LINE --------------------->>

# code to generate weight for t+1 using all prior t
# first prediction is first month end in testing set
for i in range(len(df_test)):

    df_this = equalise_weights(df_returns[n_train+i-1:n_train+i])
    df_this['month_end'] = df_test.loc[i, 'month_end']

    df_weights = pd.concat(objs=[df_weights, df_this], ignore_index=True)

# <<--------------------- YOUR CODE GOES ABOVE THIS LINE --------------------->>
```
for R

```r
# YOUR CODE GOES BELOW THIS LINE ------------------------------------------
  
  # This is your playground. Delete/modify any of the code here and replace with 
  # your methodology. Below we provide a simple, naive estimation to illustrate 
  # how we think you should go about structuring your submission and your comments:
  
  # data <- bind_rows(training_data, test_data)
  data <- test_data
  
  # We use the Inverse Volatility Weighting (https://en.wikipedia.org/wiki/Inverse-variance_weighting) strategy as our approach.
  data_long <- 
    data %>% 
    pivot_longer(
      contains("Stock"), 
      names_to = "stock", 
      values_to = "value"
    ) # pivot data to long format (columns: month_end, stock, value)
  
  weight <- 
    data_long %>% 
    summarise(vol = sd(value), .by = stock) %>% # calculate standard deviation (volatility) for each stocks' returns
    mutate(
      inv_vol = 1/vol,              # calculate the inverse of the volatility                          
      tot_inv_vol = sum(inv_vol),   # create a new column = the sum of the volatilities
      weight = inv_vol/tot_inv_vol  # create a weight column
    )
  
  # We need to convert the structure of the weight dataframe back to a wide format with dates
  data_out <- 
    data_long %>% 
    left_join(weight, by = "stock") %>% 
    select(month_end, stock, weight) %>% 
    pivot_wider(names_from = stock, values_from = weight)
    
# YOUR CODE GOES ABOVE THIS LINE ------------------------------------------
```

## The Output

Once you have the function generating the weights you can use the `plot_total_return` to compare the index and your portfolio output. Please keep the output consistent so that you don't have to recode the plotting function.

