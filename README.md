# ptradingstretegy

#Algorithm trading approach that exploits the price relationship between correlated assets 

#1.Select and validate a pair
-choose two assets with strong historical correlation ( eg. two stocks in the same sector like KO and PEP )


import yfinance as  yf 
from statsmodels.tsa.stattools import  coint 

#Download data from coca-cola (KO) and PepsiCo (PEP)

stock1 = yf.download ("KO" , start = "2020-01-01" , end = "2025-02-23" )["Close"]
stock2 = yf.download ("PEP" , start = "2020-01-01" , end = "2025-02-23" ) ["Close"]

#Cointegration test 

score, p_value , _ = coint (stock1 , stock2)
print (f"Cointegration p-value : {p_value :.4f}") # p< 0.05 suggests cointegration

#2. Calculate the spread 
- Compute spread between two assets
- Use a simple difference or a hedge ratio (e.g from a linear regression) to normalize the relationship

  import statsmodels.api as sm
  #Fit a linear regression to find the hedge ratio

  X = sm.add_constant (stock2) #PEP as independet variable
  model = sm.OLS (stock1 , X ) . fit () # KO as independent variable
  hedge_ratio = model.params[1]

  #Calculate spread
  spread = stock1 - hedge_ratio  *  stocks2

  #3.Define Trading Thresholds

  -Determine mean and standard deviation of the spread .
  -Set entry /exit points (eg, buy the spread when it's below mean -2 6 , sell when above mean +2 6)

  #Rolling stastitics for the spread

  spread_mean = spread.rolling(window=20).mean()
  spread_std = spread.rolling(window =20).std()
  z-score = (spread - spread_mean) /spread_std
  
  #Thresholds
  upper_threshold = 2.0
  lower_threshold = -2.0

  
  #4.Generate Trading signals
  - Trade the spread : go long(buy stock1 , sell stock2)when the spread is undervalued
  -  go short (sell stock1 , buy stock2 ) when overvalued , exit when spread reverts
 
    import pandas as pd
   
    #Create DataFrame for signals
    data = pd.DataFrame({"Spread" : spread , "Z_Score" : z_score}).dropna()

    #Signals : 1 = long spread , -1 = short spread , 0 = exit
    data ["Signal"] = 0
    data.loc[data["Z_Score"] < lower_threshold , "Signal"] =1 #Long spread
    data.loc[data ["Z_Score"] > upper_threshold , "Signal"] = -1 # short spread
    data ["position"] = data ["Signal"].shift(1)

  #5. Backtest and Execute
  -Calculate return based on the positions in both assets , then backtest the strategy
  - If viable, deploy with real time data
    #Returns for each stocks
    data ["Return1"] = stock1.pct_change()
    data ["Return2"] = stock2.pct_change()

    #Strategy returns (Long stock1, short stock2 when position =1 , reverse when - 1)
    data ["Strategy_Returns " ] = data ["position"] * (data ["Return1"] - hedge_ratio *
    data ["Return2"])
    cumulative_returns = (1 + data ["Strategy_Returns"]).cumprod()
    print(f"Cumulative Return :  {Cumulative_returns [-1] : 2f}")


    #pseudo-code for live trading
    #latest_spread  = get_latest_spread ("KO" , "PEP"  , hedge_ratio)
    #Z_score = (latest_spread - mean )/ std
    #if Z_score <-2:
    #  buy ("KO") : sell ("PEP")
    #elif Z_score >2 :
    #  sell("KO"): buy("PEP")  
    
    
    

  
  



##
