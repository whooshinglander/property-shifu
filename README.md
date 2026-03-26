# Property AI Toolkit
### Built for Singapore property professionals

Two AI tools that work together.

| Tool | What it does |
|---|---|
| **RAG** | Your AI that answers questions about your listings and documents |
| **HAT** | Your AI that analyses the Singapore market and finds historical patterns |

---

## RAG — Your Property Knowledge Base

Your client asks: *"Does this condo allow pets?"* or *"What's the maintenance fee at The Sail?"*

Instead of digging through PDFs, you ask your AI. It reads all your brochures, listings, and contracts — and answers instantly, with the source document cited.

**What you feed it:**
- Property brochures (PDF)
- Listing fact sheets
- CEA standard contracts
- Developer prospectuses
- Any FAQs you've written for clients

---

## HAT — Your Singapore Market Analyst

You ask: *"Is now a good time to buy in District 15?"* or *"How does this market compare to 2018?"*

The HAT loads 20+ years of URA and HDB data, finds the historical periods that looked most like today — same interest rates, same transaction volume, same price momentum — and tells you what happened to prices in the 6 and 12 months that followed.

**Example questions it can answer:**
- "We're seeing ABSD cooling and rising SORA — when did we last see this, and what happened?"
- "Volume is down 20% but prices are flat — is this a buying window or a warning?"
- "District 9 prices up 8% YoY — is this the start of a run or the top?"

---

## What you need to prepare

**For RAG:**
- Your PDFs and brochures — drop them in a folder, the AI reads them all

**For HAT:**
- URA REALIS data export (free, register at ura.gov.sg/maps/api)
- Or: HDB resale data from data.gov.sg (free, direct download)
- Your developer handles the rest

---

## Getting started

Send this repo to your developer or AI agent. The file `AGENT.md` has complete build instructions.

**You don't need to understand the code.** Your job is collecting the documents and data. Their job is making it run.

---

## What you'll end up with

- Ask any question about any property in your documents — instant answer with source
- Ask about market timing — get historical analogs and what happened next
- Runs privately on your own machine
- Near-zero cost (under SGD 10/month, or free with a local model)

---

*Built on open-source tools. Singapore data sources. No proprietary lock-in.*
