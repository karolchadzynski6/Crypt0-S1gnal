# Crypt0-S1gnal
Bulding AI course project

# Project Title

Final project for the Building AI course

## Summary

(this prgram have been already used as my final assigment od IT2 course)
CryptoSignal – Bitcoin Market Analysis and Alert System

CryptoSignal is a Python-based application that monitors the Bitcoin market in real time. The program collects market data from cryptocurrency exchanges, calculates technical indicators such as RSI, MACD, and SMA, and sends trading alerts to a Telegram group when predefined conditions are met.


## Background

The cryptocurrency market operates 24/7 and is highly volatile. Traders often miss important price movements because it is impossible to monitor charts continuously.

My motivation comes from my interest in financial markets and programming. I wanted to create a tool that automatically analyzes market conditions and informs users about potential trading opportunities.

This topic is important because cryptocurrencies are becoming increasingly popular, and many investors need tools that help them make faster and more informed decisions.

## How is it used?

The solution is designed for:

cryptocurrency traders,
beginner investors,
people interested in technical analysis.

The program runs automatically in the background. Every minute it downloads fresh market data, calculates indicators, evaluates market conditions, and sends notifications to a Telegram group.

Users receive alerts without constantly watching trading charts.

MY CODE:
```
import ccxt
import pandas as pd
from ta.momentum import RSIIndicator
from ta.trend import MACD, SMAIndicator
from datetime import datetime
import asyncio
import time
from telegram import Bot
import csv
import os

# =========================
# TELEGRAM SETTINGS
# =========================

TELEGRAM_TOKEN = 'YOUR_BOT_TOKEN'
CHAT_ID = 'YOUR_CHAT_ID'

bot = Bot(token=TELEGRAM_TOKEN)

# =========================
# CSV LOGGING SETUP
# =========================

CSV_FILE = 'alerts_log.csv'

# Create CSV file if it does not exist
if not os.path.exists(CSV_FILE):
    with open(CSV_FILE, mode='w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow([
            'timestamp',
            'symbol',
            'timeframe',
            'signal',
            'details'
        ])

# =========================
# LOG ALERT TO CSV
# =========================

def log_alert_to_csv(symbol, timeframe, signal, details):
    with open(CSV_FILE, mode='a', newline='') as file:
        writer = csv.writer(file)

        writer.writerow([
            datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S'),
            symbol,
            timeframe,
            signal,
            details
        ])

# =========================
# FETCH DATA AND CALCULATE
# INDICATORS
# =========================

def fetch_and_analyze():
    try:
        exchange = ccxt.binance({
            'enableRateLimit': True
        })

        ohlcv = exchange.fetch_ohlcv(
            'BTC/USDT',
            timeframe='1m',
            limit=100
        )

        df = pd.DataFrame(
            ohlcv,
            columns=[
                'timestamp',
                'open',
                'high',
                'low',
                'close',
                'volume'
            ]
        )

        df['timestamp'] = pd.to_datetime(
            df['timestamp'],
            unit='ms'
        )

        # Calculate RSI
        df['RSI'] = RSIIndicator(
            close=df['close'],
            window=14
        ).rsi()

        # Calculate MACD
        macd = MACD(close=df['close'])
        df['MACD'] = macd.macd()
        df['MACD_signal'] = macd.macd_signal()

        # Calculate SMA
        df['SMA_20'] = SMAIndicator(
            close=df['close'],
            window=20
        ).sma_indicator()

        return df

    except Exception as e:
        print(f"❌ Error fetching data: {e}")
        return None

# =========================
# SEND TELEGRAM ALERT
# =========================

async def send_alert(message):
    try:
        await bot.send_message(
            chat_id=CHAT_ID,
            text=message
        )

        print(f"✅ ALERT SENT: {message}")

    except Exception as e:
        print(f"❌ Telegram error: {e}")

# =========================
# MAIN PROGRAM LOOP
# =========================

print("🔄 Bot started. Checking every 1 minute...")

last_signal = None

while True:

    df = fetch_and_analyze()

    if df is not None:

        last = df.iloc[-1]

        message = None
        signal_type = None

        # RSI signals

        if last['RSI'] > 70:
            message = (
                f"⚠️ RSI = {last['RSI']:.2f} "
                f"– Overbought (BTC/USDT, 1m)"
            )
            signal_type = "RSI Overbought"

        elif last['RSI'] < 30:
            message = (
                f"📈 RSI = {last['RSI']:.2f} "
                f"– Oversold (BTC/USDT, 1m)"
            )
            signal_type = "RSI Oversold"

        # MACD bullish crossover

        elif (
            last['MACD'] > last['MACD_signal']
            and
            df['MACD'].iloc[-2] <= df['MACD_signal'].iloc[-2]
        ):
            message = (
                "📊 MACD crossed above signal line "
                "– Possible BUY (1m)"
            )
            signal_type = "MACD Bullish Cross"

        # MACD bearish crossover

        elif (
            last['MACD'] < last['MACD_signal']
            and
            df['MACD'].iloc[-2] >= df['MACD_signal'].iloc[-2]
        ):
            message = (
                "📉 MACD crossed below signal line "
                "– Possible SELL (1m)"
            )
            signal_type = "MACD Bearish Cross"

        # Send and log alert only if signal changed

        if message and message != last_signal:

            try:
                loop = asyncio.get_running_loop()

            except RuntimeError:
                loop = asyncio.new_event_loop()
                asyncio.set_event_loop(loop)

            if loop.is_running():
                asyncio.create_task(
                    send_alert(message)
                )
            else:
                loop.run_until_complete(
                    send_alert(message)
                )

            log_alert_to_csv(
                symbol='BTC/USDT',
                timeframe='1m',
                signal=signal_type,
                details=message
            )

            last_signal = message

        else:
            print("ℹ️ No new signal.")

    else:
        print("⚠️ Data not available.")

    # Wait 60 seconds before next analysis

    time.sleep(60)
```


## Data sources and AI methods

The project uses historical and real-time market data for the BTC/USDT trading pair obtained through exchange APIs.

The system calculates several technical indicators:

Relative Strength Index (RSI)
Moving Average Convergence Divergence (MACD)
Simple Moving Average (SMA)

Although the current version uses rule-based analysis rather than advanced machine learning, AI techniques could be integrated in the future. For example:

price trend prediction using machine learning,
anomaly detection,
sentiment analysis based on social media and news.

The application is implemented in Python and processes new market data every minute.

## Challenges

The project has several limitations:

technical indicators do not guarantee profitable trades,
cryptocurrency markets can change unexpectedly due to news or global events,
false signals may occur,
the system currently analyzes only one cryptocurrency pair (BTC/USDT),
no automatic trading is performed.

Therefore, the tool should support decision-making rather than replace human judgment.

## What next?

Future improvements could include:

support for multiple cryptocurrencies,
integration of machine learning models for price prediction,
sentiment analysis of social media and news,
web dashboard with market visualization,
mobile application,
automatic portfolio management,
backtesting strategies on historical data.

These features could transform the project into a complete intelligent trading assistant.


## Acknowledgments

This project was developed using:

Python programming language,
Binance API,
Telegram Bot API,
Pandas library for data analysis,
TA-Lib / technical analysis libraries (if used),
educational materials and online documentation related to algorithmic trading and technical analysis.
