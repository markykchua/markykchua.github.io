---
title: "Diversification - A Double-Edged Sword in Investment "
date: 2024-04-28
tags: ["stock market", "sp500", "diversification"]
draft: false
---


# Diversity: A Double-Edged Sword in Investment 
When examining the performance of the S&P 500, one encounters the complex interplay of diversity as both a boon and a burden. While diversity can diversify risk and potentially enhance returns, it also presents challenges in identifying consistently outperforming companies.


```python
import os 
import pandas as pd

import datetime as dt
import yfinance as yf

# read the sp500 constituents
tickers = pd.read_csv('data/constituents.csv')['Symbol'].to_list()

def normalize_to_day_1(df: pd.DataFrame):
    """
    Normalize stock prices to day 1 for a given DataFrame. If there is no data or only one data point, return the original DataFrame
    
    Args:
    df (pd.DataFrame): DataFrame containing stock price data.
    
    Returns:
    pd.DataFrame: DataFrame with stock prices normalized to day 1.
    """
    if df.shape[0] > 1: # check if there is data
        df['Close'] = df['Close'] / df['Close'].iloc[0]
    return df

```


```python
def download_data_to_local(tickers, years_back=40):
    """
    Download historical stock price data for a list of tickers and save to local CSV files.
    
    Args:
    tickers (list): List of stock ticker symbols.
    years_back (int): Number of years to trace back in history (default is 40).
    """
    # Create a directory if it doesn't exist to store downloaded data
    if not os.path.exists('yfin_data'):
        os.makedirs('yfin_data')
    
    for ticker in tickers:
        # Define the file name for the CSV file
        file_name = f'yfin_data/{ticker}.csv'
        
        # Check if the file already exists
        if not os.path.isfile(file_name):
            try:
                # Download historical stock price data from Yahoo Finance
                df = yf.download(ticker, dt.datetime.today() - dt.timedelta(365 * years_back), dt.datetime.today())['Close']
                
                # Save the data to a local CSV file
                df.to_csv(file_name)
                print(f"Data downloaded and saved for {ticker}.")
            except Exception as e:
                print(f"Error downloading data for {ticker}: {e}")
        else:
            print(f"Data already exists for {ticker}.")

# download all data to local 
download_data_to_local(tickers)
download_data_to_local(['SPY'])
```


```python
def performance_against_spy(years):
    start_date = dt.date.today() - dt.timedelta(365*years) # dt.date(2001, 1, 1)
    end_date = dt.date.today() # dt.date(2001, 9, 1) # is the the right range? 

    gain_dict = {}

    for target in tickers:
        try: 
            target_data = pd.read_csv(f'yfin_data/{target}.csv')
            if target_data.shape[0] < 1: # there is no data
                continue

            baseline_data = pd.read_csv('yfin_data/SPY.csv')

            target_data = target_data.loc[(target_data['Date'] >= str(start_date)) & (target_data['Date'] <= str(end_date))]
            baseline_data = baseline_data.loc[(baseline_data['Date'] >= target_data['Date'].iloc[0]) & (baseline_data['Date'] <= target_data['Date'].iloc[-1])]

            target_data = normalize_to_day_1(target_data)
            baseline_data = normalize_to_day_1(baseline_data)

            baseline_data = baseline_data.reset_index(drop=True)
            target_data = target_data.reset_index(drop=True)
            result = target_data['Close']/baseline_data['Close']
            gain_dict[target] = [result.describe()['mean'], result.iloc[-1]]
        except Exception as e: 
            print(f'Error processing data for {target}: {e}')

    return pd.DataFrame.from_dict(gain_dict, columns=['mean_gain', 'final_gain'], orient='index')
```

# Performance Comparison Over 10 Years

In this analysis, we evaluate the performance of each company relative to a benchmark over a consistent 10-year timeframe. We adopt a strategy of investing the entirety of the portfolio on day 1 and holding without any further trades throughout the entire duration (buy and hold strategy).

Two key metrics are utilized to gauge performance:

- Average Gain (mean_gain): This metric represents the average value of gains observed over the entire 10-year period. It provides insight into the sustained performance trend exhibited by each company relative to the benchmark.

- Final Gain (final_gain): The final gain metric signifies the cumulative gain or loss at the conclusion of the 10-year period. It offers a snapshot of the total investment return achieved by each company compared to the benchmark.

By assessing both the average and final gains, we gain a comprehensive understanding of each company's performance trajectory and its deviation from the benchmark over the specified time horizon.


```python
ten_years = performance_against_spy(10)
ten_years.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean_gain</th>
      <th>final_gain</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>498.000000</td>
      <td>498.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>1.232946</td>
      <td>1.373975</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.889522</td>
      <td>3.351409</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.337732</td>
      <td>0.076777</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.786708</td>
      <td>0.503544</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1.017769</td>
      <td>0.863344</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1.409650</td>
      <td>1.487220</td>
    </tr>
    <tr>
      <th>max</th>
      <td>12.786836</td>
      <td>70.025228</td>
    </tr>
  </tbody>
</table>
</div>



# Top 10 winners in the last 10 years


```python
ten_years.sort_values('final_gain', ascending=False).head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean_gain</th>
      <th>final_gain</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>NVDA</th>
      <td>12.786836</td>
      <td>70.025228</td>
    </tr>
    <tr>
      <th>AMD</th>
      <td>5.700903</td>
      <td>13.886379</td>
    </tr>
    <tr>
      <th>AVGO</th>
      <td>3.132011</td>
      <td>7.844226</td>
    </tr>
    <tr>
      <th>AXON</th>
      <td>2.992951</td>
      <td>7.489256</td>
    </tr>
    <tr>
      <th>ANET</th>
      <td>2.995974</td>
      <td>7.400234</td>
    </tr>
    <tr>
      <th>FICO</th>
      <td>3.210805</td>
      <td>7.277456</td>
    </tr>
    <tr>
      <th>MPWR</th>
      <td>3.293648</td>
      <td>6.835737</td>
    </tr>
    <tr>
      <th>CDNS</th>
      <td>3.077830</td>
      <td>6.746862</td>
    </tr>
    <tr>
      <th>LRCX</th>
      <td>2.828337</td>
      <td>5.994500</td>
    </tr>
    <tr>
      <th>DXCM</th>
      <td>3.756735</td>
      <td>5.627210</td>
    </tr>
  </tbody>
</table>
</div>



## Winners: mean gain vs final gain


```python
ten_years.sort_values('mean_gain', ascending=False).head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean_gain</th>
      <th>final_gain</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>NVDA</th>
      <td>12.786836</td>
      <td>70.025228</td>
    </tr>
    <tr>
      <th>PAYC</th>
      <td>6.429954</td>
      <td>4.466564</td>
    </tr>
    <tr>
      <th>AMD</th>
      <td>5.700903</td>
      <td>13.886379</td>
    </tr>
    <tr>
      <th>CZR</th>
      <td>5.427620</td>
      <td>3.297652</td>
    </tr>
    <tr>
      <th>ENPH</th>
      <td>4.581057</td>
      <td>5.574516</td>
    </tr>
    <tr>
      <th>MRNA</th>
      <td>4.128266</td>
      <td>3.010234</td>
    </tr>
    <tr>
      <th>DXCM</th>
      <td>3.756735</td>
      <td>5.627210</td>
    </tr>
    <tr>
      <th>TSLA</th>
      <td>3.513252</td>
      <td>4.502813</td>
    </tr>
    <tr>
      <th>MSCI</th>
      <td>3.492788</td>
      <td>4.319004</td>
    </tr>
    <tr>
      <th>NFLX</th>
      <td>3.438502</td>
      <td>4.325752</td>
    </tr>
  </tbody>
</table>
</div>



# So what? 
Consider this: if you had invested in NVDA ten years ago and managed to hold it until the date of the writing, you outperform the market by a staggering 70 times, it's an impressive feat. However, it's crucial to note that this success hinges on selecting the right investment. This highlights the double-edged nature of diversity – while it can potentially lead to significant gains, it also underscores the importance of making informed and strategic choices.

# The intersection of the sets obtained from the mean gain and final gain calculations. 
This gives us the companies that appear in the top 10 lists for both mean gain and final gain, indicating consistency in performance across the entire 10-year period.


```python
set(ten_years.sort_values('mean_gain', ascending=False).head(10).index.to_list()).intersection(set(ten_years.sort_values('final_gain', ascending=False).head(10).index.to_list()))
```




    {'AMD', 'DXCM', 'NVDA'}



3 tickers out of 10. Only 3 being very consistent. 

# Top 10 losers in the last 10 years
How about the bottom performers over the past decade? Let's take a look at the top 10 losers."


```python
ten_years.sort_values('final_gain').head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean_gain</th>
      <th>final_gain</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>PARA</th>
      <td>0.531836</td>
      <td>0.076777</td>
    </tr>
    <tr>
      <th>WBD</th>
      <td>0.455530</td>
      <td>0.077259</td>
    </tr>
    <tr>
      <th>VFC</th>
      <td>0.714683</td>
      <td>0.081301</td>
    </tr>
    <tr>
      <th>VTRS</th>
      <td>0.446383</td>
      <td>0.086191</td>
    </tr>
    <tr>
      <th>WBA</th>
      <td>0.611434</td>
      <td>0.094463</td>
    </tr>
    <tr>
      <th>NWL</th>
      <td>0.663243</td>
      <td>0.096088</td>
    </tr>
    <tr>
      <th>PCG</th>
      <td>0.539582</td>
      <td>0.137929</td>
    </tr>
    <tr>
      <th>APA</th>
      <td>0.337732</td>
      <td>0.138313</td>
    </tr>
    <tr>
      <th>AAL</th>
      <td>0.608257</td>
      <td>0.141332</td>
    </tr>
    <tr>
      <th>CCL</th>
      <td>0.707907</td>
      <td>0.144535</td>
    </tr>
  </tbody>
</table>
</div>



If you purchased PARA shares a decade ago, your investment would have yielded only 7% of your initial principal, without factoring in dividends. Diversity is a blessing. 

## Losers - mean gain vs final gain


```python
ten_years.sort_values('mean_gain').head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean_gain</th>
      <th>final_gain</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>APA</th>
      <td>0.337732</td>
      <td>0.138313</td>
    </tr>
    <tr>
      <th>MRO</th>
      <td>0.342048</td>
      <td>0.283311</td>
    </tr>
    <tr>
      <th>EQT</th>
      <td>0.355710</td>
      <td>0.253020</td>
    </tr>
    <tr>
      <th>TRGP</th>
      <td>0.367153</td>
      <td>0.405555</td>
    </tr>
    <tr>
      <th>WYNN</th>
      <td>0.373051</td>
      <td>0.171937</td>
    </tr>
    <tr>
      <th>BKR</th>
      <td>0.380943</td>
      <td>0.175998</td>
    </tr>
    <tr>
      <th>DVN</th>
      <td>0.392236</td>
      <td>0.279455</td>
    </tr>
    <tr>
      <th>HAL</th>
      <td>0.397285</td>
      <td>0.227000</td>
    </tr>
    <tr>
      <th>FCX</th>
      <td>0.399859</td>
      <td>0.547621</td>
    </tr>
    <tr>
      <th>SLB</th>
      <td>0.404176</td>
      <td>0.180858</td>
    </tr>
  </tbody>
</table>
</div>




```python
set(ten_years.sort_values('final_gain').head(10).index.to_list()).intersection(set(ten_years.sort_values('mean_gain').head(10).index.to_list()))
```




    {'APA'}



Interestingly, within the group of underperformers, there's a significant disparity between the mean and final gains.

# Would there be winners consistently? 
Could there be consistent winners? Over varying timeframes, from 10 years down to 1 year, let's investigate if there are companies that consistently appear in the top 10 list.


```python
top_mean_gainer = set(tickers)
for i in range(10, 1, -1):
    top10 = set(performance_against_spy(i).sort_values('mean_gain', ascending=False).head(10).index)
    common_winners = top10.intersection(top_mean_gainer)
    if len(common_winners) > 1:
        top_mean_gainer = common_winners
    else:
        print(f"The consistent winners across {i} years:")
        break
top_mean_gainer
```

    The consistent winners across 3 years:





    {'ENPH', 'MRNA', 'TSLA'}



Only three companies consistently appear across all analyzed timeframes. However, it's worth noting that if the timeframe is less than 3 years, no such companies are identified.

# Summary 
Diversity is both a blessing and a curse. 

What are your thoughts on this analysis?

What aspects would you like to explore further?

Feel free to reach out with any additional thoughts or questions!


```python
!jupyter nbconvert --to markdown diviersification_or_not.ipynb
```

    [NbConvertApp] Converting notebook diviersification_or_not.ipynb to markdown
    [NbConvertApp] Writing 31163 bytes to diviersification_or_not.md

