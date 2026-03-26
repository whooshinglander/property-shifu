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

You ask: *"My client wants to buy in the OCR. Is now a good time?"*

Shifu answers:
```
Singapore Property Market — OCR Analysis | March 2026

CURRENT CONDITIONS (Source: URA Q4 2025 release, Jan 2026)
- Private residential PPI: +3.4% for full year 2025 (slowest since 2020)
- OCR prices: +1.0% in Q4 2025 (holding up vs CCR which fell -3.2%)
- OCR avg resale PSF 2025: $1,532 (up from $1,460 in 2024)
- New supply: 9,185 GLS units in H1 2026 pipeline (50% above 10-year avg)
- SORA (3M compounded): ~3.4% → effective home loan rate ~4.2-4.4%
- URA caution: "households should exercise prudence given uncertain macroeconomic outlook"

POPULATION DEMAND SIGNAL
Singapore's total population reached 6.04 million in 2024. New citizens and PRs
continue to enter the market. However, DPM Gan Kim Yong (Committee of Supply,
Feb 2026) noted citizen population growth has slowed to 0.7% — structural demand
is real but moderating.

65% of 2026 new launches will be in OCR (Tengah, Tampines, Bayshore, Dairy Farm).
Increased supply = more negotiating power for buyers vs 2024.

CLOSEST HISTORICAL MATCHES
  #1 | Q3 2018  [prices +3.2% YoY, vol slowing, ABSD just hiked]
     → OCR prices 12 months later: -2.1%
     → What happened: July 2018 ABSD hike (SC 12%→15%) cooled market
     → Key diff vs today: ABSD already at 20% for SC — harder to hike further

  #2 | Q2 2014  [prices +3.8% YoY, vol -9%, cooling measures in place]
     → OCR prices 12 months later: flat (-0.3%)
     → What happened: market absorbed cooling measures, found floor in 2015-16

  #3 | Q3 2012  [prices +4.1% YoY, rates low, HDB upgrader wave active]
     → OCR prices 12 months later: +5.4%
     → What happened: BTO completions triggered upgrader wave into private market

SHIFU'S READ (March 2026)
The 2014 analog is most relevant — prices holding but not running, supply
coming online, cooling measures already in place. History says flat to slight
softening over 12 months.

BUT: 2026 OCR has one variable 2014 didn't — BTO completions from 2019-2021
launches will reach MOP in 2024-2026, releasing a wave of upgrader demand.
National Development Minister Chee Hong Tat: "55,000 BTO flats to be launched
2025-2027" — MOP demand wave is real and 5 years out.

BOTTOM LINE FOR YOUR CLIENT:
Rates are near their peak and likely to ease in 2026. Supply is rising short-term
but upgrader demand is structurally strong for 3-5 years. If they're buying to
hold 5+ years, the data supports buying. If they need to sell in 2 years, wait.
```

Your client sees you've done the research. No other agent is walking in with this.

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

Send this repo to your developer or AI coding agent (e.g. Claude, Cursor, Copilot). The file `AGENT.md` has complete build instructions.

**Note on the word "agent":** in this repo, *agent* means your AI coding assistant or developer — not a property agent. When Shifu talks about *your clients*, it means property buyers and sellers. When it talks about *you*, it means you the property agent (also called broker — same thing).

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
