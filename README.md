# Mini Project 3 — Agentic RAG on Social-Media

A small, reproducible **RAG** demo over a **Kaggle emotion-labeled tweets** dataset.  
Answers natural-language questions with **grounded, cited examples** from the corpus.

## What’s here
- **Report:** `reports/report.md` (CAPB format: context, data, MVP architecture, bakeoff, eval, risks)
- **Code:** minimal pipeline — ingest → embed → retrieve → generate (with citations)
- **Data:** script expects a local Kaggle CSV → converts to `data/processed/posts.jsonl`

## Quick start
```bash
# setup
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# prepare data (convert Kaggle CSV to JSONL)
python src/data/clean.py  --in data/raw/tweets.csv  --out data/processed/posts.jsonl

# build vector index (Chroma + e5-small)
python src/index/build_chroma.py  --in data/processed/posts.jsonl  --db .chroma

# evaluate baseline (15 questions; reports metrics)
python src/eval/evaluate.py  --db .chroma  --k 5
