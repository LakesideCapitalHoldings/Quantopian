import talib
import numpy as np
import pandas as pd
import math
import datetime
import pytz

def initialize(context):

    set_universe(universe.DollarVolumeUniverse(99, 100))

    context.LOW_RSI = 30
    context.HIGH_RSI = 70

    context.Rebalance_Days = 1

    context.rebalance_date = None
    context.rebalance_hour_start = 10
    context.rebalance_hour_end = 15

    context.simplerating = 0

    set_commission(commission.PerShare(cost=0.03))
    set_slippage(slippage.VolumeShareSlippage(volume_limit=0.25, price_impact=0.1))
    #makes algorithm trade minutely
    schedule_function(func = rebalance,
                      date_rule = date_rules.every_day(),
                      time_rule = time_rules.market_open())

def MACD(prices, fastperiod=12, slowperiod=26, signalperiod=9):

    macd, signal, hist = talib.MACD(prices,
                                    fastperiod=fastperiod,
                                    slowperiod=slowperiod,
                                    signalperiod=signalperiod)
    return macd[-1] - signal[-1]

def rebalance_trade(context, data, exchange_time):

    # Only rebalance if we are in the user specified rebalance time-of-day window
    if exchange_time.hour < context.rebalance_hour_start or exchange_time.hour > context.rebalance_hour_end:
       return

    context.rebalance_date = exchange_time


def simple_rating(average_price50, average_price20, context, stock):
    prices = history(40, '1d', 'price')
    context.simplerating = 0
    macd = prices.apply(MACD, fastperiod=12, slowperiod=26, signalperiod=9)
    rsi = prices.apply(talib.RSI, timeperiod=14).iloc[-1]

    if rsi[stock] < context.LOW_RSI:
        context.simplerating += 1
    if macd[stock] > 0:
        context.simplerating += 1
    #if average_price50 < average_price20:
        #context.simplerating += 1
        
    return context.simplerating

def order_stocks(NetRtg, average_price50, average_price20, context, current_position, current_price, stock, volatility, weights):
    
    if NetRtg < 0 or NetRtg > 0 or NetRtg == 0:

        #if simple_rating(average_price50, average_price20, context, stock) == 0 and NetRtg < 0:
            #order_target_percent(stock, weights[0], stop_price=current_price + volatility / 2)
            #log.info('Short: %s' % stock)

        if simple_rating(average_price50, average_price20, context, stock) == 1 and NetRtg > 0: 
            order_target_percent(stock, weights[1], stop_price=current_price - volatility / 2)
            log.info('Buy: %s' % stock)

        if simple_rating(average_price50, average_price20, context, stock) == 2 and NetRtg > 0: 
            order_target_percent(stock, weights[2], stop_price=current_price - volatility / 2)
            log.info('DoubleBuy: %s' % stock)

    else:
        pass

def rebalance(context, data):
        # Get the current exchange time, in the exchange timezone
    exchange_time = pd.Timestamp(get_datetime()).tz_convert('US/Eastern')

    # If it's a rebalance day (defined in initialize()) then rebalance:
    if  context.rebalance_date == None or exchange_time > context.rebalance_date + datetime.timedelta(days=context.Rebalance_Days):
        
        rebalance_trade(context, data, exchange_time)
        
    for stock in data:

        current_position = context.portfolio.positions[stock].amount
        current_price = data[stock].price
        average_price50 = data[stock].mavg(50)
        average_price20 = data[stock].mavg(20)
        volatility = data[stock].stddev(5)
        prices = history(40, '1d', 'price')

        macd = prices.apply(MACD, fastperiod=12, slowperiod=26, signalperiod=9)
        rsi = prices.apply(talib.RSI, timeperiod=14).iloc[-1]

        NetRtg = ((50-rsi[stock])/1000 + macd[stock]/50)*2
                               
        weights = [-abs(NetRtg), abs(NetRtg), abs(2*NetRtg)]
                   
        order_stocks(NetRtg, average_price50, average_price20, context, current_position, current_price, stock, volatility,
              weights)
        
        #print "NetRtg: " + str(NetRtg)
        #print "simple rating: " + str(context.simplerating)

def handle_data(context, data):
    pass

