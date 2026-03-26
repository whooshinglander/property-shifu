# Property AI Toolkit — RAG + HAT

Two AI tools for property professionals, built on free and open-source components.

| Tool | What it does | Who uses it |
|---|---|---|
| **RAG** | Answer questions about specific properties from documents | Buyers, tenants, agents looking up listings |
| **HAT** | Analyse market conditions, find historical analogs, advise on timing | Brokers, investors, advisors |

Both can be built independently. Start with whichever fits your immediate need.

---

## Prerequisites

- Python 3.10+
- pip
- Basic comfort with the terminal
- No paid APIs required for the core build

---

# PART 1 — RAG (Property Knowledge Base)

## What it does

A buyer asks: *"Does Condo X allow pets?"* or *"What's the maintenance fee at Block Y?"*

The RAG searches your documents (listings, brochures, contracts, FAQs), finds the relevant section, and answers. No more digging through PDFs manually.

## How it works

```
Your documents → chunked into passages → embedded as vectors → stored in a local DB
User question → embedded → similarity search → top passages retrieved → LLM answers
```

## Step 1 — Install dependencies

```bash
pip install langchain chromadb sentence-transformers openai pypdf python-docx
```

- `chromadb` — local vector database, free, runs on your machine
- `sentence-transformers` — free embedding model, no API key needed
- `openai` — for the answering step (or swap for a free local model)
- `pypdf` / `python-docx` — read PDF and Word documents

## Step 2 — Prepare your documents

Organise your documents in a folder:

```
/documents/
    listings/
        condo-x-brochure.pdf
        block-y-factsheet.pdf
    contracts/
        standard-tenancy.pdf
    faqs/
        buyer-guide.pdf
```

Any format works — PDF, Word, plain text. The more documents, the better the answers.

## Step 3 — Build the knowledge base

Create `build_rag.py`:

```python
from langchain.document_loaders import PyPDFDirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import Chroma

# Load all documents
loader = PyPDFDirectoryLoader("./documents/")
documents = loader.load()

# Split into chunks (500 chars, 50 overlap)
splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = splitter.split_documents(documents)
print(f"Created {len(chunks)} chunks from {len(documents)} documents")

# Embed and store locally (free, no API key)
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
db = Chroma.from_documents(chunks, embeddings, persist_directory="./chroma_db")
db.persist()
print("Knowledge base built. Run query.py to ask questions.")
```

Run it:
```bash
python build_rag.py
```

This runs once. Re-run whenever you add new documents.

## Step 4 — Ask questions

Create `query.py`:

```python
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import Chroma
from langchain.chat_models import ChatOpenAI
from langchain.chains import RetrievalQA
import os

# Load the knowledge base
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
db = Chroma(persist_directory="./chroma_db", embedding_function=embeddings)

# Set up the QA chain
llm = ChatOpenAI(
    api_key=os.environ["OPENAI_API_KEY"],
    model="gpt-4o-mini"  # cheap, fast, good enough
)
qa = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=db.as_retriever(search_kwargs={"k": 3}),
    return_source_documents=True
)

# Ask a question
question = input("Ask about a property: ")
result = qa({"query": question})
print("\nAnswer:", result["result"])
print("\nSources:")
for doc in result["source_documents"]:
    print(f"  - {doc.metadata.get('source', 'unknown')} (page {doc.metadata.get('page', '?')})")
```

Run it:
```bash
export OPENAI_API_KEY=your_key_here
python query.py
```

## Step 5 — Keep it updated

When you get new listings or documents:
1. Add them to `/documents/`
2. Re-run `python build_rag.py`
3. The new content is immediately searchable

## Free alternative (no OpenAI key)

Replace the LLM with a local model using Ollama:

```bash
# Install Ollama (ollama.ai)
ollama pull llama3.2
```

```python
# In query.py, replace ChatOpenAI with:
from langchain.llms import Ollama
llm = Ollama(model="llama3.2")
```

Runs entirely on your machine. No API costs. Slightly slower.

---

# PART 2 — HAT (Property Market Analyst)

## What it does

A broker asks: *"Is now a good time to buy in District 15?"* or *"How does this market compare to 2018?"*

The HAT loads historical property data, finds periods that match current conditions, shows what happened to prices next, and explains the pattern.

**HAT = Heuristic Anchored Thinking.** It's RAG for numerical/time-series data instead of documents.

## How it works

```
Historical market data → normalised feature vectors → stored as parquet
Current conditions → feature vector → cosine similarity search → top matching periods
"What happened next" → forward returns from those periods → pattern summary
```

## Step 1 — Install dependencies

```bash
pip install pandas pyarrow requests yfinance scikit-learn numpy
```

## Step 2 — Define your core features

Choose 4-6 features that define the market regime. More than 6 dilutes the signal.

**Recommended for property:**
1. Mortgage rate (or central bank rate)
2. Transaction volume (monthly, YoY change)
3. Price index (YoY change)
4. Days on market (average)
5. New listings vs absorption rate
6. Price-to-income ratio (if available)

**Rule:** if you want to add a 7th, make it a context feature instead — pulled on demand, not used for matching.

## Step 3 — Fetch historical data

Create `fetch_history.py`:

```python
"""
HAT — Property | fetch_history.py
Fetches historical property market data and saves to history.parquet.
Edit DATA_SOURCES to point to your local CSV exports or APIs.
"""

import pandas as pd
import requests
import io
import os

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
OUTPUT_PATH = os.path.join(SCRIPT_DIR, "history.parquet")

# ── Option A: Load from local CSV files ──────────────────────────────────────
# Export your historical data to CSV (from URA, HDB, PropNex, ERA, etc.)
# Columns needed: date, mortgage_rate, transaction_volume, price_index,
#                 days_on_market, new_listings, absorption_rate

def load_from_csv():
    df = pd.read_csv("./data/market_history.csv", index_col=0, parse_dates=True)
    return df

# ── Option B: Singapore URA API (free, no key needed) ────────────────────────
def fetch_ura_price_index():
    url = "https://www.ura.gov.sg/uraDataService/invokeUraDS?service=PMI_Resi_Rental_Median"
    # URA requires an access key — register free at: https://www.ura.gov.sg/maps/api/
    # Replace with your key
    headers = {"AccessKey": "YOUR_URA_KEY"}
    r = requests.get(url, headers=headers)
    return r.json()

# ── Option C: Use FRED for mortgage rates (free) ─────────────────────────────
def fetch_fred(series_id, start="2000-01-01"):
    url = f"https://fred.stlouisfed.org/graph/fredgraph.csv?id={series_id}"
    r = requests.get(url, timeout=30)
    df = pd.read_csv(io.StringIO(r.text), index_col=0, parse_dates=True)
    df.columns = [series_id]
    df[series_id] = pd.to_numeric(df[series_id], errors="coerce")
    return df[start:]

def build_history():
    print("Building property market history...")

    # Fetch mortgage/interest rate from FRED
    # Singapore: use MAS data or SORA. US: use MORTGAGE30US.
    mortgage = fetch_fred("MORTGAGE30US")  # replace with local rate series

    # Load your transaction/price data
    # Replace this with your actual data source
    try:
        local_data = load_from_csv()
        df = pd.concat([mortgage, local_data], axis=1)
    except FileNotFoundError:
        print("No local CSV found. Using mortgage rate only as placeholder.")
        df = mortgage

    df = df.ffill()
    df.to_parquet(OUTPUT_PATH)
    print(f"Saved {len(df)} rows to {OUTPUT_PATH}")
    return df

if __name__ == "__main__":
    build_history()
```

**Where to get property data:**
- **Singapore:** URA API (free, register at ura.gov.sg/maps/api), HDB resale data (data.gov.sg)
- **Malaysia:** NAPIC (free, napic.jpph.gov.my)
- **General:** Export from PropNex, ERA, or any CRM to CSV

## Step 4 — Snapshot current conditions

Create `snapshot.py`:

```python
"""
HAT — Property | snapshot.py
Captures current market conditions and saves to snapshot.json.
Update this weekly or monthly with fresh data.
"""

import json
import os
from datetime import datetime

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
OUTPUT_PATH = os.path.join(SCRIPT_DIR, "snapshot.json")

def capture_snapshot():
    # Fill these in manually or from your data source
    snapshot = {
        "as_of": datetime.today().strftime("%Y-%m"),
        "market": "Singapore District 15",  # edit per market
        "conditions": {
            "mortgage_rate": 3.5,           # current rate %
            "price_index_yoy": 4.2,         # YoY price change %
            "transaction_volume_yoy": -8.5, # YoY volume change %
            "days_on_market": 32,           # average days
            "absorption_rate": 0.68,        # listings sold / new listings
            "price_to_income": 14.2,        # median price / median annual income
        },
        "notes": "Post-cooling measures. HDB upgraders active. Interest rate plateau."
    }

    with open(OUTPUT_PATH, "w") as f:
        json.dump(snapshot, f, indent=2)
    print(f"Snapshot saved to {OUTPUT_PATH}")
    return snapshot

if __name__ == "__main__":
    capture_snapshot()
```

## Step 5 — Find historical analogs

Create `match.py`:

```python
"""
HAT — Property | match.py
Finds historical periods most similar to current market conditions.
"""

import pandas as pd
import numpy as np
import json
import os

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
HISTORY_PATH = os.path.join(SCRIPT_DIR, "history.parquet")
SNAPSHOT_PATH = os.path.join(SCRIPT_DIR, "snapshot.json")
OUTPUT_PATH = os.path.join(SCRIPT_DIR, "analogs.json")

# Core features for matching — edit to match your data columns
FEATURES = ["mortgage_rate", "price_index_yoy", "transaction_volume_yoy",
            "days_on_market", "absorption_rate"]

def cosine_similarity(a, b):
    a, b = np.array(a, dtype=float), np.array(b, dtype=float)
    norm_a, norm_b = np.linalg.norm(a), np.linalg.norm(b)
    if norm_a == 0 or norm_b == 0:
        return 0.0
    return float(np.dot(a, b) / (norm_a * norm_b))

def find_analogs(top_n=5):
    df = pd.read_parquet(HISTORY_PATH)
    with open(SNAPSHOT_PATH) as f:
        snapshot = json.load(f)

    current = snapshot["conditions"]
    current_vector = [current.get(f, 0) for f in FEATURES if f in df.columns]
    available = [f for f in FEATURES if f in df.columns]

    df_feat = df[available].dropna()
    means, stds = df_feat.mean(), df_feat.std()
    stds[stds == 0] = 1

    df_norm = (df_feat - means) / stds
    current_norm = [(current.get(f, 0) - means[f]) / stds[f] for f in available]

    # Exclude last 6 months
    cutoff = pd.Timestamp.today() - pd.DateOffset(months=6)
    df_hist = df_norm[df_norm.index < cutoff]

    similarities = df_hist.apply(
        lambda row: cosine_similarity(current_norm, row.values), axis=1
    )

    top_days = similarities.nlargest(top_n * 5)
    episodes = []
    used = set()

    for date, sim in top_days.items():
        if date in used:
            continue

        ep_end = date + pd.DateOffset(months=1)
        used.update(pd.date_range(date - pd.DateOffset(months=1), ep_end, freq="MS"))

        # What happened to prices 6 and 12 months after
        future_6m = df.get("price_index_yoy", pd.Series())
        future_slice = future_6m[ep_end: ep_end + pd.DateOffset(months=12)]

        fwd_6m = round(float(future_slice.iloc[5]), 2) if len(future_slice) > 5 else None
        fwd_12m = round(float(future_slice.iloc[11]), 2) if len(future_slice) > 11 else None

        ep_data = df[date: ep_end]
        ep_macro = {f: round(float(ep_data[f].mean()), 2)
                    for f in available if f in ep_data.columns}

        episodes.append({
            "rank": len(episodes) + 1,
            "similarity": round(sim, 4),
            "period": date.strftime("%Y-%m"),
            "conditions": ep_macro,
            "forward_returns": {"price_yoy_6m": fwd_6m, "price_yoy_12m": fwd_12m}
        })

        if len(episodes) >= top_n:
            break

    output = {
        "as_of": snapshot["as_of"],
        "market": snapshot["market"],
        "current_conditions": current,
        "top_analogs": episodes
    }

    with open(OUTPUT_PATH, "w") as f:
        json.dump(output, f, indent=2)

    # Print summary
    print(f"\nHAT — Property Market Analogs for {snapshot['market']}")
    print(f"As of: {snapshot['as_of']}\n")
    for ep in episodes:
        fwd = ep["forward_returns"]
        print(f"  #{ep['rank']} | {ep['period']} | similarity: {ep['similarity']}")
        if fwd.get("price_yoy_6m") is not None:
            print(f"       Prices 6m later: {fwd['price_yoy_6m']:+.1f}% YoY")
        if fwd.get("price_yoy_12m") is not None:
            print(f"       Prices 12m later: {fwd['price_yoy_12m']:+.1f}% YoY")
        print()

    return output

if __name__ == "__main__":
    find_analogs()
```

## Step 6 — Create HAT.md

Seed your HAT's memory file:

```markdown
# HAT — Property Market
_Accumulated insight. Read before any market timing discussion._

## Market covered
[Your market — e.g. Singapore District 9-15, Johor Bahru, KL Mont Kiara]

## Core features used for matching
1. Mortgage rate
2. Price index YoY
3. Transaction volume YoY
4. Days on market
5. Absorption rate

## Context features (pull on demand)
- Cooling measures in effect
- Developer launch pipeline
- Foreigner buying activity
- HDB upgrader activity (SG specific)
- Rental yield vs mortgage rate spread

## Key frameworks
- **Affordability ceiling:** when mortgage rate × price-to-income > threshold, buyers exit
- **Supply lag:** new supply takes 3-5 years to hit market — today's en bloc = 2028 supply
- **Cooling measure cycle:** SG government intervenes when prices rise >8% YoY, removes when volume drops >30%

## Sessions
[Add notes after each market discussion]
```

## Step 7 — Run it

```bash
python fetch_history.py    # one-time setup
python snapshot.py         # update monthly
python match.py            # find analogs anytime
```

---

# Folder Structure

```
property-ai/
    rag/
        documents/          ← your PDFs and brochures
        chroma_db/          ← auto-generated vector store
        build_rag.py
        query.py
    hat/
        data/               ← your historical CSV exports
        history.parquet     ← auto-generated
        snapshot.json       ← auto-generated
        analogs.json        ← auto-generated
        HAT.md              ← your accumulated insight
        fetch_history.py
        snapshot.py
        match.py
    README.md
```

---

# Decision Guide

**Use RAG when:**
- Client asks about a specific property
- You need to search across many documents fast
- Answering "what does the contract say about X"

**Use HAT when:**
- Advising on market timing
- Client asks "should I buy now or wait"
- Comparing current market to historical cycles
- Understanding where we are in the property cycle

**Use both together:**
- RAG answers the "what" (property details)
- HAT answers the "when" (market timing)

---

# Cost

| Component | Cost |
|---|---|
| ChromaDB (vector store) | Free, local |
| Sentence Transformers (embeddings) | Free, local |
| OpenAI gpt-4o-mini (answering) | ~$0.001 per question |
| Ollama local LLM (free alternative) | Free, runs on your machine |
| FRED data | Free |
| URA/HDB data | Free (Singapore) |

Total for a small brokerage: under $10/month if using OpenAI. Free if using Ollama locally.

---

_Built by WhooshingLander. HAT pattern — Heuristic Anchored Thinking._
