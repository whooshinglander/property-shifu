# BUILD.md — Build Instructions for Shifu
### Singapore market. Non-technical property agent. One unified AI advisor.

---

## Who is using this — read before you build anything

The person using Shifu is a Singapore property agent. CEA licensed. Works deals daily — WhatsApp-heavy, client meetings, viewings. Not technical. Will never read this file.

**How he actually works:**
- Gets a new listing → needs to prep for a client meeting in 2 hours
- Client asks "should I buy now or wait?" — he needs a real answer, not "it depends"
- Shares everything on WhatsApp — factsheets, analysis, comparables
- Competes against other agents who have access to the same listings. His edge is preparation and data.

**What Shifu must feel like to him:**
- **Fast.** He's between viewings. 2-minute interaction max for a factsheet.
- **Confident.** Output should read like a senior analyst wrote it, not a chatbot.
- **WhatsApp-ready.** Every output should be copy-pasteable directly into a client message.
- **Singapore-specific.** URA data, SORA rates, district numbers, CEA compliance — not generic global property advice.
- **No jargon he has to translate.** Plain English. If you must use a term, define it inline.

**What Shifu must never do:**
- Give generic answers ("it depends on your financial situation")
- Hallucinate transaction prices — if URA data isn't loaded yet, say "data pending, loading URA comparables"
- Confuse "agent" (property agent = the user) with "agent" (AI coding tool = you)
- Output walls of text. Every response must be scannable in 30 seconds on a phone screen.
- Give unsolicited financial advice. Shifu is a research tool. The agent advises. The agent is responsible.

**The test before shipping:** show the output to someone who is not technical. Could they read it between viewings on their phone and immediately know what to tell a client? If no — simplify.

---

## Instructions for the AI agent building this

You are an AI coding agent. You are reading this file to build Shifu. Follow these rules exactly.

**Before writing a single line of code:**
1. Read this entire file top to bottom
2. Read `README.md` — understand what the end user expects
3. Read `PLB_LEARNINGS.md` — understand the presentation formula Shifu must follow
4. Read `FACTSHEET_TEMPLATE.md` — this is the mandatory output format

**When building:**
- Build incrementally. Get the URA data scraper working first. Confirm data is loading before building the chat interface.
- Test every scraper with real API calls before moving on. Do not mock data.
- If a URA endpoint returns an error, log it clearly and try the data.gov.sg fallback. Do not silently fail.
- Every output Shifu generates must follow the format in `FACTSHEET_TEMPLATE.md` and the PLB formula in `PLB_LEARNINGS.md`. Do not invent a different format.
- Do not hallucinate property prices. If URA data is not loaded, Shifu must say "URA data not yet loaded — run fetch_history.py first."

**When writing Shifu's system prompt:**
- Shifu speaks directly to a property agent, not a developer
- Shifu's tone: confident, data-backed, direct. Not chatty. Not hedging.
- Shifu always asks for clarification when "agent" is ambiguous
- Shifu never gives financial advice — it gives research. The agent advises the client.
- Shifu's outputs are WhatsApp-ready by default — clean, scannable, no markdown headers

**When you are done:**
- Confirm every data source is loading (URA API, data.gov.sg HDB, MAS SORA)
- Run one test factsheet query and one market timing query
- Paste the outputs in a `TEST_OUTPUT.md` file so the property agent can see what Shifu looks like before using it

---

Shifu is a single AI advisor with two capabilities:
1. **Listing intelligence** — reads the broker's property documents and outputs selling points, comparables, and client-ready summaries
2. **Market analysis** — loads Singapore historical market data, finds historical analogs, advises on timing

The broker talks to Shifu. Shifu handles both. The technical split (RAG for documents, HAT for market data) is invisible to the end user.

**Goal:** make the broker look like the most prepared, data-backed agent in the room. Every time.

**Default market:** Singapore private residential + HDB resale.
**Default data sources:** URA REALIS, data.gov.sg, MAS SORA.
**Default language:** English.
**CEA compliance note:** Shifu provides data-driven research only — it does not give regulated financial or property advice. The broker remains responsible for all client recommendations.

**Terminology note:** "property agent" and "property broker" are interchangeable in Singapore. Shifu should use "property agent" as default (CEA-recognised term) but accept both. When the broker uses the word "agent" ambiguously (could mean property agent OR AI agent), Shifu must ask: *"Do you mean your client's agent, or an AI agent?"* Never assume.

---

## How Shifu should respond to listing queries

When the broker asks about a listing, Shifu should NOT just answer the question asked. It should proactively output:
1. Top selling points ranked by buyer appeal
2. Price vs district average (good value or not)
3. Recent comparable transactions (pulled from URA data)
4. Estimated rental yield
5. Ideal buyer profile
6. One-liner the broker can say to the client

This is the format that makes the broker look prepared. Build the prompt template around this output, not plain Q&A.

---

## What Shifu learned from PropertyLimBrothers

PLB is Singapore's most successful property YouTube channel (79,600 subscribers, 5,000+ videos, top videos 400K+ views). They are a creative agency + realty combined. These are the lessons Shifu is built on.

**Lesson 1: Feeling first, data second**
PLB leads every video with lifestyle — the dream of living there — before specs and price. Shifu must do the same. When outputting listing research, lead with a lifestyle headline before the numbers.

❌ Wrong: "3-bedroom, 1,227 sqft, $2,088,000, D20"
✅ Right: "Resort-style living with unblocked greenery — rare space in mature Bishan estate"

**Lesson 2: The PLB title formula = the Shifu headline formula**
Every PLB video title hits every buyer filter in one line:
`[Name] - [Bedrooms] with [sqft] in [District] | $[Price] | [Agent]`

Shifu should generate this headline for every listing. It is the factsheet title, the WhatsApp message opener, the email subject line.

**Lesson 3: Data without narrative is a spreadsheet**
PLB combines URA data with case studies and stakes. "Why some condo buyers WIN in 2026 (and others OVERPAY BADLY)." The data gives credibility, the stakes keep attention.

Shifu must add one line of context to every number:
❌ "PSF: $1,701"
✅ "$1,701 psf — 8% below D20 average of $1,850. Room to grow."

**Lesson 4: The one number they can't Google**
PLB developed the "Disparity Effect" — their proprietary analysis of price gaps between comparable properties. It gives them unique insight buyers can't find elsewhere.

Shifu's version: always include one non-obvious data point. Last 3 transaction prices for this exact development. Gap between this unit's PSF and the district ceiling. Something that makes the broker feel like they have insider knowledge.

**Lesson 5: Personal brand on everything**
Every PLB video names the agent. Not just "PLB." The agent becomes associated with results.

Shifu must always output an agent card at the end of every factsheet — name, CEA number, specialisation, one-line track record. The data builds trust in the market. The agent card builds trust in the person.

**Lesson 6: Consistency beats perfection**
5,000 videos. Same format every time. PLB built a production system, not a production house.

Shifu's factsheet must be completeable in 20 minutes. If it takes 3 hours, it won't be used. The template is non-negotiable — every field, every time, same order.

---

## Shifu's Listing Output Format (mandatory)

Every time Shifu is asked about a listing, the output must follow this structure exactly:

```
[LIFESTYLE HEADLINE]
[Development] · D[XX] · [Tenure] · $[Price]

─────────────────────────────
FAST FACTS
─────────────────────────────
Bedrooms:    [X-Bedroom + Study]
Size:        [X,XXX] sqft
PSF:         $[X,XXX] — [X]% [below/above] D[XX] average
Floor:       [X] of [XX] · [Facing] · [View]
Tenure:      [Freehold / 99-yr from XXXX]
MRT:         [Station] ([Line]) — [X] min walk
Maintenance: $[XXX]/month

─────────────────────────────
WHY THIS PROPERTY
─────────────────────────────
[3-5 bullet points. Each = one specific, data-backed reason.]
• Value: $X,XXX psf — below recent transactions of $X,XXX–$X,XXX psf
• [Unique feature + why it matters to buyer]
• [School / MRT / amenity + specific distance]
• Rental: est. $X,XXX–$X,XXX/month → X.X% gross yield
• [One non-obvious insight the buyer can't Google]

Recent comparable transactions:
  #[floor]-[stack]: $[price] ($[psf] psf) — [Month Year]
  #[floor]-[stack]: $[price] ($[psf] psf) — [Month Year]
  #[floor]-[stack]: $[price] ($[psf] psf) — [Month Year]

─────────────────────────────
IDEAL BUYER
─────────────────────────────
[2 sentences. Who this is for. Why it fits their life.]

─────────────────────────────
WHAT TO TELL YOUR CLIENT
─────────────────────────────
"[One sentence. Punchy. Data-backed. The line that closes.]"

─────────────────────────────
[AGENT NAME] · CEA [RXXXXXXXX]
[X] transactions closed · D[XX] specialist
+65 [XXXX XXXX]
─────────────────────────────
```

This format is not optional. Every field, every time. If data is unavailable, state "data pending" — never leave blank or skip the section.

---

## What Shifu asks when the agent wants a factsheet

When the property agent says "create a factsheet" or "help me with a listing," Shifu must ask for what it needs — don't generate a half-empty output. Use this exact prompt sequence:

```
Shifu: I'll prepare your factsheet. Give me what you have and I'll fill in the rest from URA data.

Required (you must provide):
1. Development name
2. Unit type (e.g. 3-bedroom + study)
3. Size in sqft
4. Asking price
5. Floor and stack (e.g. #08-12) — needed for view and comparable lookup
6. Your name and CEA licence number

Optional (Shifu will estimate if missing):
- Facing direction
- Current monthly maintenance fee
- Any recent renovations or unique features
- Who you think the ideal buyer is

Once you provide these, Shifu will generate the full factsheet including:
- PSF vs district average (from URA data)
- Last 3 comparable transactions in the same development
- Estimated rental yield
- Ideal buyer profile
- The one-liner to use with your client
```

If the agent provides partial info, Shifu generates what it can and marks missing fields as "data pending — please provide [specific field]."

---

## Overview

Two tools to build:

1. **RAG** — document-based Q&A. Broker feeds it PDFs/brochures, clients or agents ask questions, it retrieves answers from the documents.
2. **HAT (Heuristic Anchored Thinking)** — quantitative market analog finder. Takes current market conditions, finds the most similar historical periods using cosine similarity, returns what happened to prices next.

Both are standalone Python scripts. No web framework required unless the broker wants a UI later.

---

## Stack

- Python 3.10+
- ChromaDB (local vector store — no cloud, no API key)
- sentence-transformers (free embeddings, runs locally)
- OpenAI gpt-4o-mini for answering (or Ollama for fully local/free)
- pandas + pyarrow for HAT data layer
- numpy for similarity computation
- FRED API (free, no key needed for basic series)

---

## Part 1 — RAG

### Install

```bash
pip install langchain chromadb sentence-transformers openai pypdf python-docx
```

### File structure

```
rag/
    documents/          ← broker drops PDFs here
    chroma_db/          ← auto-generated, don't touch
    build_rag.py        ← run once per batch of new documents
    query.py            ← run to ask questions
```

### build_rag.py

```python
from langchain.document_loaders import PyPDFDirectoryLoader, DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import Chroma
import os

DOCS_PATH = "./documents/"
DB_PATH = "./chroma_db/"

print("Loading documents...")
loader = PyPDFDirectoryLoader(DOCS_PATH)
documents = loader.load()
print(f"Loaded {len(documents)} pages from {DOCS_PATH}")

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", ".", " "]
)
chunks = splitter.split_documents(documents)
print(f"Split into {len(chunks)} chunks")

embeddings = HuggingFaceEmbeddings(
    model_name="all-MiniLM-L6-v2",
    model_kwargs={"device": "cpu"}
)

db = Chroma.from_documents(
    chunks,
    embeddings,
    persist_directory=DB_PATH,
    collection_metadata={"hnsw:space": "cosine"}
)
db.persist()
print(f"Knowledge base ready. {len(chunks)} chunks indexed.")
```

### query.py

```python
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import Chroma
from langchain.chat_models import ChatOpenAI
from langchain.chains import RetrievalQA
from langchain.prompts import PromptTemplate
import os

DB_PATH = "./chroma_db/"

PROMPT = PromptTemplate(
    input_variables=["context", "question"],
    template="""You are a helpful property assistant. Answer the question based only on the provided context.
If the answer isn't in the context, say "I don't have that information in my documents."
Be concise and direct.

Context:
{context}

Question: {question}

Answer:"""
)

embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2", model_kwargs={"device": "cpu"})
db = Chroma(persist_directory=DB_PATH, embedding_function=embeddings)

llm = ChatOpenAI(
    api_key=os.environ.get("OPENAI_API_KEY"),
    model="gpt-4o-mini",
    temperature=0
)

qa = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=db.as_retriever(search_kwargs={"k": 4}),
    chain_type_kwargs={"prompt": PROMPT},
    return_source_documents=True
)

print("Property AI ready. Type your question (or 'quit' to exit).\n")
while True:
    question = input("Question: ").strip()
    if question.lower() in ["quit", "exit", "q"]:
        break
    if not question:
        continue

    result = qa({"query": question})
    print(f"\nAnswer: {result['result']}")
    print("\nSources:")
    seen = set()
    for doc in result["source_documents"]:
        src = doc.metadata.get("source", "unknown")
        page = doc.metadata.get("page", "?")
        key = f"{src}:{page}"
        if key not in seen:
            seen.add(key)
            print(f"  {os.path.basename(src)}, page {page}")
    print()
```

### Free/offline alternative

Replace the OpenAI LLM block with:
```python
from langchain.llms import Ollama
llm = Ollama(model="llama3.2")  # requires Ollama installed at ollama.ai
```

### Usage for broker

```bash
# First time, and whenever new documents are added:
python build_rag.py

# Ask questions:
export OPENAI_API_KEY=sk-...
python query.py
```

---

## Part 2 — HAT

### Install

```bash
pip install pandas pyarrow numpy requests yfinance scikit-learn
```

### File structure

```
hat/
    data/                   ← broker's CSV data goes here
    history.parquet         ← auto-generated
    snapshot.json           ← auto-generated, update monthly
    analogs.json            ← auto-generated output
    SHIFU.md                  ← accumulated insight, human-readable
    fetch_history.py        ← run once + weekly refresh
    snapshot.py             ← run monthly with fresh market data
    match.py                ← run anytime to find analogs
    hat.py                  ← master runner
```

### Core matching features (6 max — do not add more)

These are the features used for cosine similarity matching. Adjust column names to match the broker's data:

1. `mortgage_rate` — prevailing home loan rate
2. `price_index_yoy` — property price index, YoY % change
3. `transaction_volume_yoy` — number of transactions, YoY % change
4. `days_on_market` — average days to sell
5. `absorption_rate` — units sold / new listings (0-1 scale)
6. `price_to_income` — median price / median annual household income

If a column isn't available, drop it from the FEATURES list. 4 features is fine. Do not pad with weak data.

### fetch_history.py

```python
"""
Fetches historical market data and builds history.parquet.
Edit DATA_SOURCES section for the broker's specific market.
"""

import pandas as pd
import requests
import io
import os

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
OUTPUT_PATH = os.path.join(SCRIPT_DIR, "history.parquet")
START = "2000-01-01"

FRED_SERIES = {
    # Singapore doesn't have FRED series — use local data sources below.
    # For reference / global context only:
    "DFF":      "fed_funds_rate",   # US Fed rate — affects SG via USD/SGD and capital flows
    "CPIAUCSL": "us_cpi",
}

# ── Singapore data sources (free) ────────────────────────────────────────────
# 1. URA Private Property Price Index (quarterly)
#    Register free: https://www.ura.gov.sg/maps/api/
#    Download: https://www.ura.gov.sg/reis/dataBrowse → Property Price Index → CSV
#    Columns needed: quarter, price_index, transaction_volume, median_psf

# 2. HDB Resale Price Index (quarterly)
#    Direct download: https://data.gov.sg/dataset/hdb-resale-price-index
#    Columns needed: quarter, index

# 3. HDB Resale Flat Prices (individual transactions, monthly)
#    Direct download: https://data.gov.sg/dataset/resale-flat-prices
#    Use this to derive volume and median prices

# 4. SORA (Singapore Overnight Rate Average) — mortgage rate proxy
#    MAS website: https://www.mas.gov.sg/monetary-policy/sora
#    Or use SOR/SIBOR historical data from MAS

# 5. Mortgage rate (3M SORA + bank spread, typically 3M SORA + 0.8-1.0%)
#    Approximate: use 3M compounded SORA + 0.9% as prevailing home loan rate

def fetch_fred(series_id, alias):
    url = f"https://fred.stlouisfed.org/graph/fredgraph.csv?id={series_id}"
    r = requests.get(url, timeout=30)
    df = pd.read_csv(io.StringIO(r.text), index_col=0, parse_dates=True)
    df.columns = [alias]
    df[alias] = pd.to_numeric(df[alias], errors="coerce")
    return df[START:]

def load_local_csv(path, date_col="date"):
    """Load broker's own CSV data. Must have a date column and numeric columns."""
    df = pd.read_csv(path, parse_dates=[date_col], index_col=date_col)
    return df[START:]

def build_history():
    print("Building history...")
    frames = []

    # FRED data
    for sid, alias in FRED_SERIES.items():
        try:
            print(f"  Fetching {sid}...")
            frames.append(fetch_fred(sid, alias))
        except Exception as e:
            print(f"  WARNING: {sid} failed: {e}")

    # Local broker data (if available)
    local_path = os.path.join(SCRIPT_DIR, "data", "market_history.csv")
    if os.path.exists(local_path):
        print(f"  Loading local data from {local_path}...")
        frames.append(load_local_csv(local_path))

    if not frames:
        print("ERROR: No data loaded. Add CSV files to hat/data/ or check FRED connection.")
        return None

    df = pd.concat(frames, axis=1, sort=True)
    df = df.ffill()  # forward-fill monthly/quarterly series

    # Derived features
    if "price_index" in df.columns:
        df["price_index_yoy"] = df["price_index"].pct_change(12) * 100
    if "transaction_volume" in df.columns:
        df["transaction_volume_yoy"] = df["transaction_volume"].pct_change(12) * 100

    # Forward returns for "what happened next"
    for col in ["price_index", "price_index_yoy"]:
        if col in df.columns:
            df[f"{col}_fwd6m"] = df[col].pct_change(6).shift(-6) * 100
            df[f"{col}_fwd12m"] = df[col].pct_change(12).shift(-12) * 100

    df.to_parquet(OUTPUT_PATH)
    print(f"\nSaved {len(df)} rows ({df.index[0].date()} to {df.index[-1].date()}) → {OUTPUT_PATH}")
    print(f"Columns: {list(df.columns)}")
    return df

if __name__ == "__main__":
    build_history()
```

### snapshot.py

```python
"""
Captures current market conditions.
The broker fills this in monthly with fresh numbers.
Or you wire it to a live data API if available.
"""

import json, os
from datetime import datetime

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
OUTPUT_PATH = os.path.join(SCRIPT_DIR, "snapshot.json")

def capture():
    snapshot = {
        "as_of": datetime.today().strftime("%Y-%m"),
        "market": "Singapore Private Residential",
        "conditions": {
            "mortgage_rate":           3.5,    # % — prevailing 3M SORA + spread (check MAS)
            "price_index_yoy":         4.2,    # % — URA PPI YoY change (check URA REALIS)
            "transaction_volume_yoy": -8.5,    # % — URA transaction volume YoY
            "days_on_market":         32,       # days — check PropNex/ERA monthly report
            "absorption_rate":         0.68,   # sold units / new launches (0-1)
            "price_to_income":         14.2,   # median condo price / median household income
        },
        "context": {
            "cooling_measures":   "ABSD: SC 20%, PR 30%, foreigner 60%. TDSR 55%.",
            "supply_pipeline":    "Edit: units launching next 12 months",
            "demand_drivers":     "Edit: e.g. HDB upgraders active, en bloc proceeds in market",
            "hdb_upgrader_cycle": "Edit: BTO completions driving upgrader demand?",
            "rental_yield":       "Edit: gross rental yield % vs mortgage rate %",
            "notes":              ""
        }
    }

    with open(OUTPUT_PATH, "w") as f:
        json.dump(snapshot, f, indent=2)

    print(f"Snapshot saved → {OUTPUT_PATH}")
    print("Edit snapshot.json to update the numbers for this month.")
    return snapshot

if __name__ == "__main__":
    capture()
```

### match.py

```python
"""
Finds historical periods most similar to current conditions.
Outputs ranked analogs with forward price returns.
"""

import pandas as pd
import numpy as np
import json, os

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
HISTORY_PATH = os.path.join(SCRIPT_DIR, "history.parquet")
SNAPSHOT_PATH = os.path.join(SCRIPT_DIR, "snapshot.json")
OUTPUT_PATH  = os.path.join(SCRIPT_DIR, "analogs.json")

# Edit to match your actual column names
FEATURES = [
    "mortgage_rate",
    "price_index_yoy",
    "transaction_volume_yoy",
    "days_on_market",
    "absorption_rate",
]

def cosine_sim(a, b):
    a, b = np.array(a, dtype=float), np.array(b, dtype=float)
    na, nb = np.linalg.norm(a), np.linalg.norm(b)
    return float(np.dot(a, b) / (na * nb)) if na and nb else 0.0

def find_analogs(top_n=5):
    df  = pd.read_parquet(HISTORY_PATH)
    with open(SNAPSHOT_PATH) as f:
        snap = json.load(f)

    current = snap["conditions"]
    available = [f for f in FEATURES if f in df.columns and current.get(f) is not None]

    if len(available) < 2:
        print("ERROR: Not enough matching columns between snapshot and history.")
        print(f"  Snapshot keys: {list(current.keys())}")
        print(f"  History columns: {list(df.columns)}")
        return None

    df_feat = df[available].dropna()
    means, stds = df_feat.mean(), df_feat.std().replace(0, 1)
    df_norm = (df_feat - means) / stds
    curr_norm = [(current[f] - means[f]) / stds[f] for f in available]

    cutoff = pd.Timestamp.today() - pd.DateOffset(months=6)
    df_hist = df_norm[df_norm.index < cutoff]
    sims = df_hist.apply(lambda r: cosine_sim(curr_norm, r.values), axis=1)

    episodes, used = [], set()
    for date, sim in sims.nlargest(top_n * 8).items():
        if date in used:
            continue
        window = pd.date_range(date - pd.DateOffset(months=3), date + pd.DateOffset(months=3), freq="MS")
        used.update(window)

        ep = df[date: date + pd.DateOffset(months=1)]
        ep_macro = {f: round(float(ep[f].mean()), 2) for f in available if f in ep.columns and ep[f].notna().any()}

        fwd = {}
        for col in ["price_index_fwd6m", "price_index_yoy_fwd6m", "price_index_fwd12m"]:
            if col in df.columns:
                val = df.loc[date, col] if date in df.index else None
                if val and not np.isnan(val):
                    fwd[col] = round(float(val), 2)

        episodes.append({
            "rank":       len(episodes) + 1,
            "similarity": round(sim, 4),
            "period":     date.strftime("%Y-%m"),
            "conditions": ep_macro,
            "forward":    fwd,
        })
        if len(episodes) >= top_n:
            break

    output = {
        "as_of":       snap["as_of"],
        "market":      snap["market"],
        "current":     current,
        "context":     snap.get("context", {}),
        "top_analogs": episodes,
    }

    with open(OUTPUT_PATH, "w") as f:
        json.dump(output, f, indent=2)

    print(f"\nHAT — {snap['market']} | {snap['as_of']}")
    print(f"Features matched: {available}\n")
    for ep in episodes:
        print(f"  #{ep['rank']} {ep['period']}  [similarity: {ep['similarity']}]")
        for k, v in ep['conditions'].items():
            print(f"       {k}: {v}")
        for k, v in ep['forward'].items():
            print(f"       → {k}: {v:+.1f}%")
        print()

    return output

if __name__ == "__main__":
    find_analogs()
```

### hat.py (master runner)

```python
"""
HAT master runner.
Bootstraps history if missing, refreshes if stale, finds analogs, prints summary.
"""

import os, sys, subprocess
SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
sys.path.insert(0, SCRIPT_DIR)

HISTORY = os.path.join(SCRIPT_DIR, "history.parquet")
SNAPSHOT = os.path.join(SCRIPT_DIR, "snapshot.json")

import pandas as pd
from datetime import date

# Bootstrap
if not os.path.exists(HISTORY):
    print("No history found. Running first-time setup...")
    import fetch_history; fetch_history.build_history()
else:
    df = pd.read_parquet(HISTORY)
    last = df.index[-1].date()
    if (date.today() - last).days > 30:
        print(f"History stale ({last}). Refreshing...")
        import fetch_history; fetch_history.build_history()
    else:
        print(f"History OK ({last}, {len(df):,} rows)")

# Snapshot
if not os.path.exists(SNAPSHOT):
    print("No snapshot found. Creating template...")
    import snapshot; snapshot.capture()
    print("Edit hat/snapshot.json with current market data, then re-run hat.py")
    sys.exit(0)

# Match
import match
output = match.find_analogs(top_n=5)

# Read SHIFU.md for context
hat_md = os.path.join(SCRIPT_DIR, "SHIFU.md")
if os.path.exists(hat_md):
    print(f"\nHAT memory: {hat_md}")
    print("Read SHIFU.md for accumulated market insight before interpreting analogs.")

print("\nShifu ready.")
```

---

## Singapore Data Sources — What to Scrape and How

All sources below are free. Build scrapers for each. Priority order: URA API first (structured, real-time), data.gov.sg second (bulk historical), MAS third (rates).

---

### Source 1 — URA API (primary, real-time)
**Register free:** https://www.ura.gov.sg/maps/api/
**Dev portal:** https://www.developer.tech.gov.sg/products/categories/data-and-apis/ura-apis/overview

Once registered you get an access key. All endpoints return JSON.

| Endpoint | What it returns | Use for |
|---|---|---|
| `PMI_Resi_Transaction` | Individual private property transactions (address, PSF, price, date, type) | Comparable transactions, PSF by development |
| `PMI_Resi_Rental_Median` | Median rental prices by district and flat type | Rental yield computation |
| `PMI_Resi_Rental_Contract` | Individual rental contracts | Granular rental data |
| `PMI_Resi_Launch` | New launch data (units launched, sold) | Supply pipeline |
| `PMI_Resi_Sold` | Units sold by project | Absorption rate |

**Sample scraper:**
```python
import requests, json

ACCESS_KEY = "your_ura_access_key"
TOKEN_URL  = "https://www.ura.gov.sg/uraDataService/insertNewToken.action"
DATA_URL   = "https://www.ura.gov.sg/uraDataService/invokeUraDS"

def get_token():
    r = requests.get(TOKEN_URL, headers={"AccessKey": ACCESS_KEY})
    return r.json()["Result"]

def get_transactions(token, batch=1):
    """Fetch private residential transactions. Paginated — batch 1-4."""
    params = {
        "service":  "PMI_Resi_Transaction",
        "batch":    str(batch),
    }
    r = requests.get(DATA_URL, headers={"AccessKey": ACCESS_KEY, "Token": token}, params=params)
    return r.json()

token = get_token()
data  = get_transactions(token, batch=1)
# data["Result"] = list of transactions with: project, street, x, y, area, floorRange,
#                  noOfUnits, contractDate, typeOfSale, price, propertyType, district,
#                  typeOfArea, tenure, psf, marketSegment
```

**Important:** token expires after each call. Request a new token for each API call. Rate limit: be polite, don't hammer. Fetch once daily and cache locally.

---

### Source 2 — data.gov.sg (bulk historical, no API key needed)
**Base URL:** https://data.gov.sg/api/action/datastore_search

| Dataset | Dataset ID | What it contains |
|---|---|---|
| HDB resale flat prices | `d_8b84c4ee58e3cfc0ece0d773c8ca6abc` | Every HDB resale transaction since 1990 |
| HDB resale price index | `d_b5a9ae32a0e9523249ce4afb0fc5e617` | Quarterly HDB resale price index |
| Private residential transactions (CCR) | `d_5785799d63a9da091f4e0b456291eeb8` | Quarterly CCR data |
| Private residential transactions (RCR) | Similar ID — search data.gov.sg | Quarterly RCR data |
| Private residential transactions (OCR) | Similar ID — search data.gov.sg | Quarterly OCR data |
| Consumer Price Index (SG CPI) | search "consumer price index" on data.gov.sg | Monthly CPI |

**Sample scraper:**
```python
import requests, pandas as pd

def fetch_datagov(dataset_id, limit=10000):
    url = f"https://data.gov.sg/api/action/datastore_search"
    params = {"resource_id": dataset_id, "limit": limit}
    r = requests.get(url, params=params)
    records = r.json()["result"]["records"]
    return pd.DataFrame(records)

# HDB resale prices
hdb = fetch_datagov("d_8b84c4ee58e3cfc0ece0d773c8ca6abc")
# Columns: month, town, flat_type, block, street_name, storey_range,
#          floor_area_sqm, flat_model, lease_commence_date, remaining_lease, resale_price

# For full history (>10k records), paginate:
def fetch_all(dataset_id, page_size=10000):
    offset, records = 0, []
    while True:
        url = "https://data.gov.sg/api/action/datastore_search"
        r = requests.get(url, params={"resource_id": dataset_id, "limit": page_size, "offset": offset})
        batch = r.json()["result"]["records"]
        if not batch: break
        records.extend(batch)
        offset += page_size
    return pd.DataFrame(records)
```

---

### Source 3 — MAS (SORA and interest rates)
**SORA daily rates:** https://eservices.mas.gov.sg/statistics/dir/DomesticInterestRates.aspx
**Direct CSV download:** https://eservices.mas.gov.sg/statistics/msb/export.xlsx?tableproduct=I.1

| Series | What it is | Use for |
|---|---|---|
| 3-month compounded SORA | Benchmark rate for most SG home loans | Mortgage rate proxy |
| SOR (deprecated 2023) | Old benchmark | Historical data pre-2023 |
| Prime lending rate | Major bank prime rate | Alternative mortgage proxy |

**Sample scraper:**
```python
import requests, pandas as pd, io

def fetch_sora():
    # MAS publishes SORA via their statistics API
    url = "https://eservices.mas.gov.sg/statistics/api/v1/bondsandbills/m/averageinterestrates"
    r = requests.get(url, params={"rows": 200, "from": "2019-01-01"})
    data = r.json()["result"]["records"]
    df = pd.DataFrame(data)
    return df
```

---

### Source 4 — SRX / 99.co (rental yield, days on market)
SRX and 99.co publish monthly rental and transaction data. Not a formal API but scrapeable via their public pages.

**99.co rental data:** https://www.99.co/singapore/research — export via browser, or use their published monthly reports.

**Alternative:** use URA `PMI_Resi_Rental_Median` API endpoint (Source 1) for rental data — cleaner and official.

---

### Source 5 — Government press releases (population, policy signals)
Not structured data — parse these for policy context that Shifu uses in its market narrative.

| Source | URL | What to extract |
|---|---|---|
| MND press releases | mnd.gov.sg/newsroom | Minister statements on housing supply, ABSD |
| URA quarterly releases | ura.gov.sg/Corporate/Media-Room/Media-Releases | Official PPI releases with commentary |
| MTI economic surveys | mti.gov.sg | GDP forecasts, population projections |
| DPM/Minister speeches | pmo.gov.sg/Newsroom | Population policy, housing demand signals |

**Scraper approach:** use `requests` + `BeautifulSoup` to scrape latest releases. Parse for keywords: "ABSD", "cooling measures", "BTO", "population", "supply". Store as text in `SHIFU.md` context section — Shifu reads these as qualitative signals.

**Key quotes already in Shifu's context (as of March 2026):**
- DPM Gan Kim Yong (COS debate, Feb 2026): citizen population growth slowing to 0.7%/yr
- Minister Chee Hong Tat: 55,000 BTO flats to be launched 2025-2027
- URA Q4 2025 release: PPI +3.4% full year 2025 (slowest since 2020), 9,185 GLS units H1 2026

---

### Scraping schedule (recommended cron)
| Source | Frequency | What changes |
|---|---|---|
| URA API transactions | Weekly (Sunday) | New transactions filed |
| data.gov.sg HDB | Monthly (1st) | New monthly resale data |
| MAS SORA | Daily | Rate changes |
| URA quarterly PPI | Quarterly (Jan/Apr/Jul/Oct) | Price index update |
| Government press releases | Weekly | Policy signals |

---

### Local storage after scraping
```
hat/
  data/
    ura_transactions.parquet    ← all private transactions, append-only
    hdb_resale.parquet          ← all HDB resale transactions
    price_index.csv             ← quarterly PPI (URA + HDB)
    sora.csv                    ← daily SORA rates
    press_releases.jsonl        ← one JSON object per release, append-only
```

All scraped data feeds into `history.parquet` via `fetch_history.py`.

---

## SHIFU.md — Singapore Seed Template

Create `hat/SHIFU.md` with this content and expand over time:

```markdown
# SHIFU — Singapore Property Market
_Heuristic Anchored Thinking. Read before any market timing discussion._

## Market
Singapore private residential + HDB resale.

## Core matching features
1. `mortgage_rate` — prevailing 3M SORA + bank spread
2. `price_index_yoy` — URA PPI YoY % change
3. `transaction_volume_yoy` — URA volume YoY % change
4. `days_on_market` — average days to sell
5. `absorption_rate` — units sold / new launches

## Context features (pull on demand)
- Active cooling measures (ABSD rates, TDSR, LTV limits)
- Developer launch pipeline (units launching next 12 months)
- Foreign buyer activity (% of transactions)
- HDB upgrader cycle (BTO completions → upgrader demand 5-7 years later)
- Rental yield vs mortgage rate spread (positive = yield > cost = buy signal)
- En bloc cycle (proceeds re-entering market as buying power)

## Singapore-specific frameworks
- **Cooling measure cycle:** government intervenes when prices rise >8% YoY (ABSD hikes), relaxes when volume drops >30% for 2+ quarters
- **Supply lag:** new launches take 3-5 years to complete. Today's en bloc = supply in 2028-2030.
- **HDB upgrader cycle:** BTO completions peak → MOP reached 5 years later → upgrader wave hits private market. Predictable, 5-year lead time.
- **TDSR 55% ceiling:** hard affordability cap. At current SORA + spread, calculate the income needed to service median-priced unit. When that exceeds median household income, demand structurally limited.
- **Foreign demand signal:** when ABSD for foreigners is 60%, any foreign buying despite the tax = very strong conviction. Watch for this as sentiment signal.
- **Rental yield floor:** historically SG private residential yields 2.5-3.5% gross. When yield compresses below 2.5%, prices are running ahead of fundamentals.

## Historical episodes (SG-specific)
- **1996-1998:** Asian Financial Crisis. Prices fell 40-45% peak to trough. Took 10 years to recover.
- **2003:** SARS. Sharp but short. V-shaped recovery within 18 months.
- **2007-2009:** GFC. Prices fell ~25%. Recovered faster than 1998 due to government stimulus.
- **2011-2013:** Post-GFC boom. ABSD introduced Dec 2011 to cool foreign demand.
- **2018:** ABSD hike (SC from 7% to 12%, foreigners from 15% to 20%). Volume fell 30%, prices flat.
- **2022-2023:** ABSD hike again (foreigners to 60%). Current regime.
- **2020:** COVID. Prices barely dipped — government support + low rates + limited supply.

## Sessions
[Add notes after each market discussion — date, what was discussed, what the data showed]
```

---

## Running order

```bash
# First time only
python fetch_history.py

# Monthly — update with fresh market data
python snapshot.py
# Then edit snapshot.json manually with current numbers

# Anytime — find analogs
python hat.py
```

---

## Common issues

**`ModuleNotFoundError`** — run `pip install <module_name>`

**`FileNotFoundError: documents/`** — create the folder: `mkdir documents`

**Chroma errors on rebuild** — delete `chroma_db/` folder and re-run `build_rag.py`

**History has no matching columns** — check that your CSV column names match the FEATURES list in `match.py`. Edit one to match the other.

**Empty analogs output** — your history doesn't have enough rows or the snapshot values are outside the historical range. Check with `python -c "import pandas as pd; df = pd.read_parquet('hat/history.parquet'); print(df.describe())"`

---

## Cost estimate

| Component | Cost |
|---|---|
| All Python libraries | Free |
| ChromaDB | Free, local |
| Sentence Transformers | Free, local |
| OpenAI gpt-4o-mini | ~$0.001/question |
| Ollama (local LLM) | Free |
| FRED data | Free |
| URA/HDB/NAPIC data | Free |

**Total:** under $5/month using OpenAI. Free if using Ollama.

---

*HAT pattern — Heuristic Anchored Thinking. Built by WhooshingLander.*
