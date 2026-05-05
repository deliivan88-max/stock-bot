import requests
import time
import os

ALPHA_KEY = os.getenv("ALPHA_KEY")
TOKEN = os.getenv("TOKEN")
CHAT_ID = os.getenv("CHAT_ID")

WATCHLIST = ["TSLA","NVDA","AAPL","BP.L","HSBA.L","SAP.DE"]

def send(msg):
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    requests.post(url, data={"chat_id": CHAT_ID, "text": msg})

def get_data(symbol):
    url = f"https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol={symbol}&interval=5min&apikey={ALPHA_KEY}"
    return requests.get(url).json()

def analyze(symbol):
    data = get_data(symbol)
    try:
        ts = data["Time Series (5min)"]
        times = list(ts.keys())

        latest = float(ts[times[0]]["4. close"])
        prev = float(ts[times[3]]["4. close"])

        volume_now = float(ts[times[0]]["5. volume"])
        volume_prev = float(ts[times[3]]["5. volume"])

        price_change = (latest - prev) / prev * 100
        volume_ratio = volume_now / volume_prev

        if price_change > 2 and volume_ratio > 2.5:
            send(f"🚀 {symbol} | {round(price_change,2)}% | Vol x{round(volume_ratio,1)}")
    except:
        pass

while True:
    for stock in WATCHLIST:
        analyze(stock)
    time.sleep(180)
