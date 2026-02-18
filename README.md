# Audit for Crypto Signal Groups  

My goal is to build a solution whose purpose is to audit how effective Telegram crypto signal groups are, and to document the technical side of its implementation.

The diagram below shows the logic I plan to implement. The solution will be built from components with the following roles:
- download messages from selected Telegram groups
- extract trading signals from messages using GenAI
- verify signal performance using historical exchange data
- present the signal performance results

![](img/signal-audit-high-level.png)

When building this solution, I also have some personal goals:
- build discipline and consistency by creating something regularly
- enjoy building something cool
- learn GenAI and play with it in practice

## What is trading signal?

Since in this project we will analyze how effective the signals are, it is worth saying a few words about what a signal is.

In short, a signal includes its exact date and a set of the following details: 
* what to buy and at what price
* at what price to take profit (usually more than one level)
* at what price to limit the loss
* the position direction (long/short).

![](img/signal-example.png)

Now let’s assume someone gives you advice and at 16:00 saying: _buy SOL/USDT below 82.2 and take profit (sell) as soon as it goes above 82.5, and if the price drops below 82 then sell and accept the loss_.

On the candlestick chart above from the exchange, we marked the following data:
1. the date and time of the signal — the signal timing will be key for its evaluation in backtesting
1. the price at which we should buy: 82.2 (entry price)
1. the price level where we exit the position and accept a loss (stop loss)
1. the price we are happy with and want to sell at (take profit)
1. the exact date and time when we end the trade

The signal could also be limited in time, which is not shown in the image above.

**Signal validation will simply be checking, using historical exchange data, whether a trade recommendation for a specific instrument—with the given entry, take profit, and stop loss prices—ended in profit or loss.**

## Phase 1 - Prompt engineering - testing in playground

The first step was to check whether an LLM could extract the signals. For this, I used the playground available in Azure AI Foundry. Here is exactly what I did:
* Deployed AI Foundry
* Gathered some example signals from one of the groups
* Prepared and tested multiple prompts and models
* Checked if the output was promising enough to justify investing more time in development

The result of the actions above was the following artifacts and conclusions:
* confirmation that the use case I planned for the LLM makes sense

![](img/prompt-engineering-manual-verification.png)

* system prompt that I will test later with different models, mainly to find the best speed and cost efficiency

```md
You are a message analyst who extracts trade recommendation from messages if possible.

<objective>
Your task is to return an object in JSON format, in accordance with the rules below, the structure, and the examples.
</objective>

<examples>
    <example>
        USER:
        ```
        Pump.fun Trading Signal (Swing)

        Instrument: PUMP/USDT
        My Opinion: Buy Stop (Pending Order)
        Entry price: $0.003012
        Stop loss: $0.002760
        Target: $0.003512
        Risk settings: 1%
        R. R. R: 1:2

        N. B: This pending order will be closed if not triggered in 24 hours.
        ```
        AI:
        ```
        {
            "isSignal":true,
            "trade" : {
                "baseCurrency": "PUMP",
                "quoteCurrency" :"USDT",
                "type" :"SWING"
                "targets" : [0.003512],
                "stopLoss" : 0.002760,
                "entry" : {
                    "type" : "BuyStop",
                    "price" : 0.003012
                },
                "validity" : {
                    "timeThreshold": "24h"
                }
            }
        }
        ```
    </example>
    <example>
        USER:
        ```
        InfoFi Explained: How Information Is Becoming a Tradable Asset in Web3

        The digital economy runs on attention, data, and influence — but until now, the financial rewards from these assets have largely gone to big tech companies. 

        InfoFi, short for Information Finance, is emerging as a new Web3 movement that aims to change that by turning information itself into a financial instrument.

        You can read more here: https://cryptosignals.org/blockchain/infofi-explained-how-information-is-becoming-a-tradable-asset-in-web3/
        ```
        AI:
        ```
        {
            "isSignal":false,
            "trade" : null
        }
        ```
    </example>
    <example>
        USER:
        ```
        ☀️ Brighten Your Monday with a FREE $20 + $500 Airdrop! ☀️

        🎉 Good Morning, Traders! 🎉

        Kick off your week with Bybit’s exclusive $20 bonus and a guaranteed $500 USDT Derivatives Airdrop!  

        🔗 JOIN NOW! (https://partner.bybit.com/b/55376)

        Here’s How to Get Started:  
        ✨ Sign Up using our special Bybit link.  
        ✨ Deposit $100 and trade $10 to claim your rewards.  
        ✨ Receive $20 instantly to start your week strong!  
        ✨ Complete the task here (https://partner.bybit.com/b/55376) to unlock your guaranteed $500 USDT Airdrop.  

        Extra Monday Boost:  
        ✨ Deposit $250 to unlock Lifetime VIP Crypto Signals – your ticket to expert strategies and market insights. Click here to claim free lifetime VIP access! (https://t.me/CSByBitBot_bot)

        Don’t miss out – grab it now! (https://partner.bybit.com/b/55376)
        ```
        AI:
        ```
        {
            "isSignal":false,
            "trade" : null
        }
        ```
    </example>
    <example>
        USER:
        ```
        Bitcoin (BTC) Price Prediction: BTC/USDT Heads Back Upward

        Price action has remained capped below a psychological resistance level in the Bitcoin market. Earlier attempts to break above this resistance failed, resulting in a downward retracement over two sessions. Let’s take a look at what may unfold next.

        BTC/USDT Long-Term Trend — Bullish (Daily Chart)

        Key Price Levels
        Resistance: $90,000, $92,500, $95,000
        Support: $89,000, $88,000, $87,000

        The Bitcoin market recorded a positive rebound in the previous session. 

        You can read more here: https://cryptosignals.org/technical-analysis/bitcoin-btc-price-prediction-btc-usdt-heads-back-upward/
        ```
        AI:
        ```
        {
            "isSignal":false,
            "trade" : null
        }
        ```
    </example>
</examples>
```


## Backlog 

### To Do
- deploy required infrastrcuture: models and db
- capture messages from telegram to db
- extract signals from messages and store them in db
- backtest signals and store results in db
- add dashboard with results
- automate solution
- automate solution deployment

### Done
- document what is signal and how it will be validated
- manually validate trading signal extraction with GenAI
- initialize repository with high-level concept description