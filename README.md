# Shifu
### Your AI property advisor. Built for Singapore brokers.

Shifu knows two things:

1. **Your listings** — ask it anything about any property in your portfolio
2. **The market** — ask it whether now is a good time to buy, sell, or wait

---

## What Shifu can do

**About your listings:**
> *"Does The Riverina allow pets?"*
> *"What's the maintenance fee at Grandeur 8?"*
> *"Show me all 3-bedroom units under $2.5M in D20"*

Shifu reads all your brochures, factsheets, and listing documents — and answers instantly.

---

**About the market:**
> *"Is D15 a good buy right now?"*
> *"How does this market compare to 2018?"*
> *"Volume is down but prices are flat — what does history say?"*

Shifu loads 20+ years of URA and HDB data, finds the historical periods that looked most like today, and tells you what happened to prices in the 6 and 12 months that followed.

---

## Example

You ask: *"D14 prices are up 5% this year but volume is dropping. Should my client buy now or wait?"*

Shifu answers:

```
Shifu — Singapore Property | March 2026

Current conditions match 3 historical periods:

  #1 | Q3 2018  [94% match]
     Prices +5.1% YoY, volume -12%, mortgage 2.1%
     → What happened: prices drifted -3.8% over the next 12 months
     → Context: ABSD hike followed 3 months later

  #2 | Q2 2014  [89% match]  
     Prices +3.8% YoY, volume -9%, mortgage 1.8%
     → What happened: prices flat at +0.4% over 12 months
     → Context: cooling measures already in place, market found floor

  #3 | Q1 2011  [85% match]
     Prices +4.5% YoY, volume -6%, mortgage 1.5%
     → What happened: prices continued +6.1% over 12 months
     → Context: pre-ABSD, upgrader demand still strong

2 out of 3 matches suggest caution. 1 suggests continued growth.
Key variable: whether another cooling measure is coming.
```

You decide what to advise. But now you're advising with data, not gut feel.

---

## What you need to set up Shifu

**For listing knowledge:**
- Your property brochures and factsheets (PDF)
- Drop them in a folder — Shifu reads them all

**For market analysis:**
- URA price index data (free at ura.gov.sg/reis)
- HDB resale data (free at data.gov.sg)
- Your developer will handle the rest

---

## Getting started

Send this repo to your developer or AI agent. The file `AGENT.md` has everything they need.

You don't need to understand the code. You just need to:
1. Collect your listing PDFs
2. Download the URA/HDB data files
3. Hand both to your developer with `AGENT.md`

Once set up, you talk to Shifu. That's it.

---

## The Shifu memory file

After each conversation, Shifu's memory file (`SHIFU.md`) is updated with what was discussed and what the data showed. Next time you ask, Shifu already knows the context.

Over time, Shifu gets smarter about your specific market, your typical client profile, and the patterns you've noticed in your own deals.

---

*Built on open-source tools. Singapore data. Free to run.*
*Inspired by PropertyLimBrothers' formula: data + story + personal brand.*
