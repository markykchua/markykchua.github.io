---
title: "Time in the market vs timing the market"
date: 2024-05-06
tags: ["stock market", "sp500", "recession"]
draft: false
---

# What is done
Use sp500 as an example to validate whether the well-known saying, "time in the market beats timing the market", makes sense or not

There have been many people making such claim such as the one on [SeekingAlpha](https://seekingalpha.com/article/4535147-time-in-the-market-beats-timing-the-market). 

Let us try to investigate whether such a claim makes sense with some assumptions and data. 

Techniques used: Monte-Carlo simulation


```python
import yfinance as yf
import datetime as dt
import pandas as pd

ticker = 'SPY'
today = dt.date.today()
s = yf.download(ticker, today - dt.timedelta(365*40), today+dt.timedelta(1))['Close']

span = 20 # suppose the investor will hold the position for span years
```

```python
# pd.DatetimeIndex, pd.Timestamp, dt.date
# pd.Timestamp is an element of pd.DatetimeIndex
def get_date_with_price(series: pd.Series, date: dt.datetime): 
    for _ in range(7): 
        if date in series.index: 
            return date
        else:
            date = date - pd.Timedelta(1, 'd')
    return date

```

```python
# time-in-the-market: passively stay in the market for a span specified. 
early_portion = s[:today - dt.timedelta(365*span)]
time_in_market_gains = []
for _ in range(1000): 
    start = early_portion.sample().index.to_pydatetime()[0]
    end = start + dt.timedelta(365*span)
    end = get_date_with_price(s, end)

    gain = (s[end] / s[start] - 1)
    time_in_market_gains.append(gain)
    
```

```python
# timing the market: you can predict the recession. you only buy 1 year after recession and hold it unti the span finish. because you may not enter the market from the beginning of the span, you time spent in the market will be less. 
import libs
recession_start_dates = [get_date_with_price(s, pd.Timestamp(recession)) for recession in libs.recession_starts] # we need a day with price data around the recession starting date

timing_market_gains = []

for _ in range(1000):
    # all-out when it is 1 year before recession; all-in when it is 1 year after recession begins. 
    start = early_portion.sample().index.to_pydatetime()[0]
    end = start + dt.timedelta(365*span)
    end = get_date_with_price(s, end)

    tmp = s[start: end]

    cash = 1000 # start with 1000
    shares = 0
    last_checked = start
    for recession in recession_start_dates: 
        if recession < end: 
            if last_checked < recession - pd.Timedelta(365, 'd'): # the recession will happen in 1 year
                if cash > 0:
                    shares = cash / tmp[last_checked] # all in on the day checked
                one_year_before_recession = get_date_with_price(tmp, recession-pd.Timedelta(365, 'd'))
                cash = shares * tmp[one_year_before_recession] # all out on the day exectaly 1 year before recession
                last_checked = one_year_before_recession

                if recession + pd.Timedelta(365, 'd') < end: # 1 year after recession, the game is still on
                    recession_annual = get_date_with_price(tmp, recession + pd.Timedelta(365, 'd'))
                    shares = cash / tmp[recession_annual] # all in on the annual of recession
                    cash = 0 
                    last_checked = recession_annual
    gain = (cash + shares*tmp[end])/1000 - 1 
    timing_market_gains.append(gain)
```


```python
result = pd.DataFrame({'time-in': time_in_market_gains, 'timing': timing_market_gains})
result.describe()
```

|       |     time-in |     timing |
|:------|------------:|-----------:|
| count | 1000        | 1000       |
| mean  |    2.37295  |    4.80538 |
| std   |    0.813132 |    1.5153  |
| min   |    0.607708 |    1.90295 |
| 25%   |    1.63796  |    3.52012 |
| 50%   |    2.40609  |    4.49298 |
| 75%   |    3.11609  |    5.97696 |
| max   |    4.02322  |    8.17598 |

```python
result.plot()
```
![results over samples](/images/time_in_the_market_vs_timing_the_market_8_1.png)


# Conclusion
If you were capable of predicting the future recession, you can roughly double your gain compared with staying in the market passively. The study here has a lot of naive assumptions. And there are a lot of questions that can be potentially interesting - for example, how much good you need to be in order to break-even the simple time-in market strategy? If you are interested, please contact me for more contents. 



