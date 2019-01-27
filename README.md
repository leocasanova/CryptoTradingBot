# CryptoTradingBot


Binome : BOTTE Alexandre & CASANOVA Leo


Arbitrage

• Retrieve the asset list of an exchange A

• Retrieve the asset list of an exchange B

• Retrieve the bid and ask price for a select currency

• Compare bidA/askB and bidB/askA

• Scale the process to execute as close to real time as possible

• Receive an email notification when a match is found

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


