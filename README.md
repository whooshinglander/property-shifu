# Shifu
### Your AI property advisor. Built for YOU.
### It will show your client that you are the best property broker and they have to choose you.

Shifu knows two things:

1. **Your listings** — ask it anything about any property and it will give you the selling points, the research
2. **The market** — ask it whether now is a good time to buy, sell, or wait

---

## What Shifu does for you

**Before a client meeting:**
> *"Tell me the selling points of The Riverina in D18"*

Shifu reads your listing documents and comes back with:
- The strongest selling points ranked by buyer appeal
- Price vs district average (is it good value?)
- Recent comparable transactions
- Estimated rental yield
- Who this property is ideal for

You walk into the meeting sounding like you've spent hours researching. You spent 2 minutes.

---

**When a client asks about timing:**
> *"My client wants to buy in D15. Is now a good time?"*

Shifu pulls 20+ years of Singapore market data, finds the historical periods that looked most like today, and tells you what happened to prices next. You give your client data-backed advice that most brokers can't.

---

## Example — Listing research

You ask: *"Give me the selling points for Grandeur 8, 3-bedroom, $2,088,000"*

Shifu answers:
```
GRANDEUR 8 — Selling Points for Client Presentation

HEADLINE: Spacious 3+Study in mature D20 estate, priced below recent transactions

TOP SELLING POINTS:
1. Value: $1,701 psf — 8% below D20 average of $1,850 psf
2. Space: 1,227 sqft — rare size for this price in D20
3. Location: Bishan MRT 8 min walk (Circle + North-South lines)
4. Schools: Raffles Institution, Catholic High within 1km
5. Rental upside: estimated $4,200–$4,500/month (2.5% yield)

RECENT COMPARABLE SALES:
- #08-12: $2,050,000 ($1,671 psf) — Mar 2026
- #11-14: $2,150,000 ($1,752 psf) — Jan 2026
This unit priced below both — room to negotiate upward at resale.

IDEAL BUYER:
HDB upgrader family with school-going children. 
Bishan location = strong rental demand from expat families (MOE schools).

ONE-LINER FOR CLIENT:
"Grandeur 8 is one of the last ways to get into Bishan below $1,750 psf — 
every comparable unit has sold higher. The data supports the price."
```

---

## Example — Market timing

You ask: *"D14 prices up 5% this year, volume dropping. Should my client buy now or wait?"*

Shifu answers:
```
Singapore Property Market — D14 Analysis | March 2026

This market matches 3 historical periods:

  #1 | Q3 2018  [94% match]
     Prices +5.1% YoY, volume -12%
     → Prices 12 months later: -3.8%
     → What happened: ABSD hike followed 3 months after

  #2 | Q2 2014  [89% match]
     Prices +3.8% YoY, volume -9%
     → Prices 12 months later: flat (+0.4%)
     → What happened: cooling measures held, market found floor

  #3 | Q1 2011  [85% match]
     Prices +4.5% YoY, volume -6%
     → Prices 12 months later: +6.1%
     → What happened: pre-ABSD, upgrader demand pushed prices higher

SHIFU'S READ:
2 of 3 matches = caution. Watch for cooling measure signals.
If no new ABSD in next 3 months, the 2011 scenario becomes more likely.
```

Your client sees you've done the research. No other broker is walking in with this.

---

## What you need

**For listing research (immediate value):**
- Your property brochures and factsheets as PDF files
- Drop them in a folder, Shifu reads them all

**For market analysis:**
- URA price index data (free — ura.gov.sg/reis)
- HDB resale data (free — data.gov.sg)

---

## Getting started

Send this repo to your developer or AI agent. The file `AGENT.md` has complete build instructions.

**Your job:**
1. Collect your listing PDFs
2. Download the URA/HDB CSV files
3. Hand everything to your developer with `AGENT.md`

Once Shifu is running, you just talk to it.

---

## Shifu gets smarter over time

Every conversation is saved to `SHIFU.md` — Shifu's memory. Next session it already knows your market, your typical client, the properties you've discussed. The longer you use it, the sharper it gets.

---

*Built on open-source Python. Singapore data (URA, HDB, MAS). Free to run.*
*Presentation formula inspired by PropertyLimBrothers.*
