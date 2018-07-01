---
id: 3721
title: Stock market analysis with R
date: 2015-12-23T10:56:29+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3721
permalink: /2015/12/23/stock-market-analysis-with-r/
dsq_thread_id:
  - "4427593317"
image: /wp-content/uploads/2015/12/R-language-logo.png
categories:
  - Blog
  - R programming language
tags:
  - computing
  - market
  - programming
  - R-language
  - scripting
  - statistics
  - stock
---
R is a programming language designed for statistical computing. I recently bought a few stocks and decided to analyse some trends based on the current stock data. So here's my first contact with R and a stock analysis. 
<!--more-->
An R analysis is composed of a bunch commands which can be drawn together to a script. The example belowsuses the quantmod libaray to download stock market data. Further I'v load the data into a chart and add indicators such as the Bollinger Band to it.

[code]
# load library
require(&quot;quantmod&quot;)
require(TTR)

# setup stock
setSymbolLookup(VW=list(name=&quot;VOW.DE&quot;, src=&quot;yahoo&quot;))

# download stock data
getSymbols(c(&quot;VW&quot;))

# create chart
chartSeries(VW, subset=&quot;last 3 months&quot;)

# add indicators
addMACD()
addBBands()
[/code]

And here is the output built by R:

<img src="https://janikvonrotz.ch/wp-content/uploads/2015/12/R-stock-analysis.png" alt="R stock analysis" width="798" height="829" class="aligncenter size-full wp-image-3741" />

The BBand shows how volatile the market or the stock is. The wider the band the higher is the volatility. The tightening of the bands is often used by technical traders as an early indication that the volatility is about to increase sharply.

I've also added the moving average convergence/divergence (MACD) which is a trend-following momentum indicator that shows the relationship between two moving averages of prices.

There are three common methods used to interpret the MACD:

1. Crossovers - As shown in the chart above, when the MACD falls below the signal line, it is a bearish signal, which indicates that it may be time to sell. Conversely, when the MACD rises above the signal line, the indicator gives a bullish signal, which suggests that the price of the asset is likely to experience upward momentum.

2. Divergence - When the security price diverges from the MACD. It signals the end of the current trend.

3. Dramatic rise - When the MACD rises dramatically (shorter moving average (grey) pulls away from the longer-term moving average (red)) that is a signal that the security is overbought and will soon return to normal levels.

Traders also watch for a move above or below the zero line because this signals the position of the short-term average relative to the long-term average.