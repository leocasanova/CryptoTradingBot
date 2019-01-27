# CryptoTradingBot


Binome : BOTTE Alexandre & CASANOVA Leo

<br/>Arbitrage

• Retrieve the asset list of an exchange A

• Retrieve the asset list of an exchange B

• Retrieve the bid and ask price for a select currency

• Compare bidA/askB and bidB/askA

• Scale the process to execute as close to real time as possible

• Receive an email notification when a match is found

<br/>CODE:

```
import numpy as np
import pandas as pd

import binance
from poloniex import Poloniex

import sched, time

import datetime

import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText


binance.set("API Key", "Secret Key")

polo = Poloniex()

s = sched.scheduler(time.time, time.sleep)


def SendNotif(comp1, comp2):
    if (comp1 == True or comp2 == True):
        fromaddr = "MYEMAIL.com"
        toaddr = "YOUREMAIL.com"
        msg = MIMEMultipart()
        msg['From'] = fromaddr
        msg['To'] = toaddr
        msg['Subject'] = "Notification for crypto LTC"
        
        if (comp1 == True and comp2 == True):
            body = "Bid (Binance) > Ask (Poloniex) AND Bid (Poloniex) > Ask (Binance)"
        elif (comp1 == True):
            body = "Bid (Binance) > Ask (Poloniex)"
        else:
            body = "Bid (Poloniex) > Ask (Binance)"
            
        msg.attach(MIMEText(body, 'plain'))
 
        server = smtplib.SMTP('smtp.gmail.com', 587)
        server.starttls()
        server.login(fromaddr, "MYPASSWORD")
        text = msg.as_string()
        server.sendmail(fromaddr, toaddr, text)
        server.quit()


def TradingBot(sc): 

    print ("Analyzing the market...")
    print('Timestamp: {:%Y-%m-%d %H:%M:%S}'.format(datetime.datetime.now()))
    print()
    
    #-------------------------------------------------------
    #Retrieve the asset list of an exchange A
    print("---------------------------------------------------------------------------")
    print()
    print("Retrieving asset list from Binance")
    pricesA = binance.tickers()
    ExA = pd.DataFrame(pricesA).transpose()
    print(ExA.head(5))  
    print("...")
    #-------------------------------------------------------
    #Retrieve the asset list of an exchange B
    print("---------------------------------------------------------------------------")
    print()
    print("Retrieving asset list from Poloniex")
    pricesB = dict(polo.returnTicker())
    ExB = pd.DataFrame.from_dict(pricesB, orient = 'index')
    print(ExB.head(5))
    print("...")
    #-------------------------------------------------------
    
    #-------------------------------------------------------
    #Retrieve the bid and ask price for a select currency
    print("---------------------------------------------------------------------------")
    print()
    print("Retrieving bid and ask price for LTC from Binance")
    LTC_A = ExA.loc['LTCBTC']
    print(LTC_A)
    print()
    
    print("---------------------------------------------------------------------------")
    print()
    print("Retrieving bid and ask price for LTC from Binance")
    LTC_B = ExB.loc['BTC_LTC']
    print(LTC_B)
    print()
    #-------------------------------------------------------
    
    #-------------------------------------------------------
    #Compare bidA/askB and bidB/askA
    print("---------------------------------------------------------------------------")
    print()
    print("Exchange A : Binance")
    print("Exchange B : Poloniex")
    print()
    print("Comparing bidA/askB and bidB/askA")
    print()
    A = (float(LTC_A['bid']), float(LTC_A['ask']))
    B = (LTC_B['highestBid'], LTC_B['lowestAsk'])
    
    A = np.array(A)
    B = np.array(B)
    
    Comp = pd.DataFrame([A,B], index = 'A B'.split(), columns = 'Bid Ask'.split())
    print(Comp)
    
    Comp1 = Comp['Bid']['A'] > Comp['Ask']['B']
    Comp2 = Comp['Bid']['B'] > Comp['Ask']['A']
    
    print()
    print("Bid A > Ask B : {}".format(Comp1))
    print("Bid B > Ask A : {}".format(Comp2))
    print()
    #-------------------------------------------------------
    
    #-------------------------------------------------------
    #Send Email Notification
    SendNotif(Comp1, Comp2)
    #-------------------------------------------------------
    
    s.enter(60, 1, TradingBot, (sc,))


s.enter(60, 1, TradingBot, (s,))
s.run()
```

<br/>Example output:

```
Analyzing the market...
Timestamp: 2019-01-27 15:55:00

---------------------------------------------------------------------------

Retrieving asset list from Binance
                ask        askQty         bid        bidQty
ETHBTC   0.03188100    1.17900000  0.03186600    0.47000000
LTCBTC   0.00904400   10.01000000  0.00904300    0.55000000
BNBBTC   0.00194450   18.13000000  0.00194390  243.18000000
NEOBTC   0.00205900  152.51000000  0.00205600  171.82000000
QTUMETH  0.01767800    3.33000000  0.01760800    1.14000000
...
---------------------------------------------------------------------------

Retrieving asset list from Poloniex
           baseVolume      high24hr    highestBid  id  isFrozen          last  \
BTC_BCN     10.122873  1.800000e-07  1.600000e-07   7         0  1.700000e-07   
BTC_BTS      5.809302  1.104000e-05  1.058000e-05  14         0  1.062000e-05   
BTC_BURST    1.850178  1.050000e-06  1.030000e-06  15         0  1.040000e-06   
BTC_CLAM     1.631277  5.021800e-04  4.637900e-04  20         0  4.638000e-04   
BTC_DASH    16.171443  2.054928e-02  1.976216e-02  24         0  1.976211e-02   

                low24hr     lowestAsk  percentChange   quoteVolume  
BTC_BCN    1.600000e-07  1.700000e-07       0.000000  5.968522e+07  
BTC_BTS    1.057000e-05  1.064000e-05      -0.003752  5.414385e+05  
BTC_BURST  1.030000e-06  1.040000e-06       0.000000  1.774618e+06  
BTC_CLAM   4.606700e-04  4.638000e-04      -0.035358  3.464030e+03  
BTC_DASH   1.975000e-02  1.986689e-02      -0.028707  8.012413e+02  
...
---------------------------------------------------------------------------

Retrieving bid and ask price for LTC from Binance
ask        0.00904400
askQty    10.01000000
bid        0.00904300
bidQty     0.55000000
Name: LTCBTC, dtype: object

---------------------------------------------------------------------------

Retrieving bid and ask price for LTC from Binance
baseVolume         157.532204
high24hr             0.009340
highestBid           0.009064
id                  50.000000
isFrozen             0.000000
last                 0.009064
low24hr              0.008996
lowestAsk            0.009065
percentChange       -0.028015
quoteVolume      17184.635085
Name: BTC_LTC, dtype: float64

---------------------------------------------------------------------------

Exchange A : Binance
Exchange B : Poloniex

Comparing bidA/askB and bidB/askA

        Bid       Ask
A  0.009043  0.009044
B  0.009064  0.009065

Bid A > Ask B : False
Bid B > Ask A : True
```


