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

## Backlog 

### To Do
- manually validate trading signal extraction with GenAI
- document what is signal and how it will be validated
- deploy required infrastrcuture: models and db
- capture messages from telegram to db
- extract signals from messages and store them in db
- backtest signals and store results in db
- add dashboard with results
- automate solution
- automate solution deployment

### Done
- initialize repository with high-level concept description