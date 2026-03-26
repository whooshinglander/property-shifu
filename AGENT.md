# AGENT.md — Build Instructions for Property AI Toolkit

These are complete instructions for building both tools. The end user is a non-technical property broker. Your job is to make both tools work with minimal friction on their end.

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
    HAT.md                  ← accumulated insight, human-readable
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
    "MORTGAGE30US": "mortgage_rate",       # US 30Y mortgage — replace for SG/MY
    "CPIAUCSL":     "cpi",
    "UNRATE":       "unemployment",
    "DFF":          "policy_rate",
}

# Singapore-specific: replace MORTGAGE30US with SORA or bank prime rate CSV
# Malaysia-specific: replace with BNM OPR data
# URA quarterly price index: https://www.ura.gov.sg/reis/dataBrowse (export to CSV)
# HDB resale: https://data.gov.sg/dataset/resale-flat-prices (export to CSV)

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
        "market": "Singapore — Edit this",
        "conditions": {
            "mortgage_rate":           3.5,    # % — current home loan rate
            "price_index_yoy":         4.2,    # % — YoY price change
            "transaction_volume_yoy": -8.5,    # % — YoY volume change
            "days_on_market":         32,       # days
            "absorption_rate":         0.68,   # 0-1, sold/new listings
            "price_to_income":         14.2,   # median price / median annual income
        },
        "context": {
            # Free text. Describe anything the numbers don't capture.
            "cooling_measures": "ABSD at 60% for foreigners",
            "supply_pipeline": "3,200 units launching Q3 2026",
            "demand_drivers": "HDB upgraders active, en bloc proceeds in market",
            "notes": ""
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

# Read HAT.md for context
hat_md = os.path.join(SCRIPT_DIR, "HAT.md")
if os.path.exists(hat_md):
    print(f"\nHAT memory: {hat_md}")
    print("Read HAT.md for accumulated market insight before interpreting analogs.")

print("\nHAT ready.")
```

---

## Data Sources by Market

### Singapore
| Data | Source | Format |
|---|---|---|
| Private property price index | URA REALIS (ura.gov.sg/reis) | CSV export |
| HDB resale prices | data.gov.sg/dataset/resale-flat-prices | CSV |
| Transaction volume | URA REALIS | CSV |
| SORA (mortgage proxy) | MAS website | CSV |
| Rental index | URA REALIS | CSV |

Register for free URA API access at: https://www.ura.gov.sg/maps/api/

### Malaysia
| Data | Source |
|---|---|
| House Price Index | NAPIC (napic.jpph.gov.my) |
| Transaction volume | NAPIC |
| OPR (mortgage proxy) | BNM (bnm.gov.my/monetary-stability) |

### Universal (FRED — free, no key)
| Series | What it is |
|---|---|
| MORTGAGE30US | US 30Y mortgage rate |
| DFF | Fed funds rate |
| CPIAUCSL | US CPI |
| UNRATE | US unemployment |

---

## HAT.md — Seed Template

Create `hat/HAT.md` with this content and expand over time:

```markdown
# HAT — Property Market
_Heuristic Anchored Thinking. Read before any market timing discussion._

## Market
[e.g. Singapore private residential, District 9-15]

## Core matching features
1. Mortgage rate
2. Price index YoY
3. Transaction volume YoY
4. Days on market
5. Absorption rate

## Context features (pull on demand)
- Active cooling measures
- Developer launch pipeline (units in next 12m)
- Foreign buyer activity
- HDB upgrader cycle (SG)
- Rental yield vs mortgage rate spread

## Key frameworks
- **Affordability ceiling:** buyers exit when monthly repayment > 30-35% gross income
- **Supply lag:** en bloc/new launches take 3-5 years to hit market
- **Cooling measure cycle (SG):** government intervenes when prices rise >8% YoY, relaxes when volume drops >30%
- **Interest rate sensitivity:** every 1% rate rise reduces max loan quantum ~10%

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
