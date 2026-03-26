# Property HAT
### A market timing tool for Singapore property brokers

HAT stands for **Heuristic Anchored Thinking**.

It does one thing: loads 20+ years of Singapore property data, finds the historical periods that looked most like today, and tells you what happened to prices next.

---

## What it actually does

You tell it the current market conditions:
- Mortgage rate: 3.5%
- Prices up 4% year-on-year
- Transaction volume down 8%
- Average days on market: 32

It searches through every quarter from 2000 to today, finds the ones that looked most similar, and shows you what happened in the 6 and 12 months that followed.

---

## Example output

```
HAT — Singapore Private Residential | March 2026

Current conditions:
  Mortgage rate: 3.5% | Prices YoY: +4.2% | Volume YoY: -8.5%
  Days on market: 32 | Absorption rate: 0.68

Top 3 historical matches:

  #1 | Q3 2018  [similarity: 94%]
       Mortgage: 2.1% | Prices: +5.1% | Volume: -12% | DOM: 35
       → Prices 6 months later:  -2.1%
       → Prices 12 months later: -3.8%

  #2 | Q2 2014  [similarity: 89%]
       Mortgage: 1.8% | Prices: +3.8% | Volume: -9% | DOM: 30
       → Prices 6 months later:  -1.5%
       → Prices 12 months later: +0.4%

  #3 | Q1 2011  [similarity: 85%]
       Mortgage: 1.5% | Prices: +4.5% | Volume: -6% | DOM: 28
       → Prices 6 months later:  +3.2%
       → Prices 12 months later: +6.1%
```

Two out of three matches showed prices softening. One showed continued growth. You decide what to do with that — but now you're deciding with data, not gut feel.

---

## What it is not

- It does not predict the future
- It does not give financial advice
- It does not replace your judgment

It gives you historical context. What you do with that context is still your call.

---

## What you need to prepare

**One CSV file** with Singapore property market history. Sources (all free):

| Data | Where to get it |
|---|---|
| URA Private Property Price Index | ura.gov.sg/reis/dataBrowse → export CSV |
| Transaction volume | Same URA REALIS page |
| HDB Resale Price Index | data.gov.sg/dataset/hdb-resale-price-index |
| SORA (mortgage rate) | mas.gov.sg/monetary-policy/sora |

You don't need all of them to start. URA price index + transaction volume is enough for a first version.

---

## What your developer needs

Hand them `AGENT.md` in this repo. It has all the code and instructions.

You just need to:
1. Download the CSV files above
2. Send them to your developer with `AGENT.md`
3. Update the current market numbers monthly (takes 5 minutes)

---

## The HAT memory file

After each time you use it, your developer (or AI agent) adds notes to a file called `HAT.md`. This is where the tool accumulates knowledge over time.

For example, after a discussion about cooling measures:

```
March 2026 — ABSD at 60% for foreigners, TDSR 55%.
Volume down 8% YoY but prices holding. 
Closest analog: Q3 2018 (post-ABSD hike). 
2018 saw prices drift -4% over 18 months before recovering.
Key difference today: supply pipeline much tighter than 2018.
```

Next time you ask a question, it reads this first. The tool gets smarter with every conversation.

---

*Built on open-source Python. Free to run. Data from URA, HDB, MAS.*
