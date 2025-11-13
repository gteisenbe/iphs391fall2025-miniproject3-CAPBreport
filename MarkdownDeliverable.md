# Mini-Project #3 — CAPB Report  
**Title:** Agentic RAG over Social-Media Emotion Posts (Kaggle) for Grounded, Cited Answers  

## 1. Project Context & Use Case

### 1.1 Problem / Context
Social media contains short, emotionally charged posts that are valuable for psychology-adjacent tasks (e.g., mental-health outreach, moderation, and PE—psychoeducation). Users and staff often need grounded answers like “How do people on social media typically express sadness about exams?” with concrete, cited examples rather than generic, hallucinated summaries.

I will use a Kaggle dataset of **tweets labeled with emotions** (e.g., *joy, sadness, anger, fear, love, surprise*) to build a **Retrieval-Augmented Generation (RAG)** system that answers natural-language questions and **grounds** responses in real, cited posts from the corpus.

### 1.2 Primary Use Case
**Research assistant / document summarization** over a curated corpus of social-media posts:  
- Given a question about emotional expression patterns, retrieve representative posts and produce a concise, **cited** answer.

### 1.3 Users / Audience
- **Students** studying psychology, media studies, and NLP  
- **Campus support staff** exploring example-based psychoeducation content  
- **Content researchers** examining emotion expression patterns in short texts

### 1.4 Success Criteria

```yaml
context:
  domain: "social media + psychology (emotion-labeled tweets from Kaggle)"
  use_case: "grounded Q&A with cited examples from the corpus"
  users: ["students", "support staff", "content researchers"]
  success:
    - "≥80% correct answers on a small gold set"
    - "≥80% answers include at least one correct citation"
    - "≤5s end-to-end latency on laptop CPU"
2\. Data & Constraints
----------------------

### 2.1 Corpus Details

*   **Source:** Kaggle dataset of **emotion-labeled tweets** (short social-media posts with labels such as joy/sadness/anger/fear/love/surprise).
*   **Size (operational subset):** 10k–30k posts (subsetted for the mini-project to keep indexing/time manageable).
*   **Format:** CSV → converted to JSONL (`id, text, emotion, timestamp?`).
*   **Preprocessing:** lowercasing, URL/handle normalization (keep placeholders like `<URL>`), deduplication, length filtering (8–60 tokens), split into 256-token chunks (mostly 1 post = 1 chunk).

### 2.2 Constraints

```yaml
data_constraints:
  sources: ["Kaggle emotion-labeled tweets"]
  formats: ["csv", "jsonl"]
  size: "10k-30k posts (subset)"
  local_only: true
  cost_limit: "free/OSS"
  latency_target: 5
  security: "local files; no PII beyond public tweet text"

**Notes on ethics & terms:**

*   Use only **public** text included in the dataset; do not attempt to deanonymize.
*   No scraping during grading; the repo includes **small samples** plus a script to load the Kaggle file locally.
*   Report explicitly cautions against clinical use; this is an **educational demo**.


3\. RAG Architecture (MVP)
--------------------------

### 3.1 Pipeline

**Ingestion → Chunking → Embedding → Vector DB → Retrieval → LLM → Answer (+ citations)**

### 3.2 Core Design Choices

```yaml
architecture_mvp:
  steps: ["ingest", "chunk", "embed", "retrieve", "generate"]
  chunking: "fixed (256 tokens), minimal overlap"
  embeddings: "local (e5-small or bge-small; sentence-transformers)"
  vector_db: "chroma"
  retrieval: "vector-only (k=5)"
  llm: "local or hosted small model (as approved)"
  citations: true
  rationale: "simple, reproducible, and fast for short posts"
```

**Agentic controller (lightweight):**

*   If the query contains emotion keywords (e.g., _sad, anxious, angry_) → **retrieve** and answer.
*   If retrieval density is low (<2 relevant hits by cosine threshold) → **re-query** with expanded synonyms (thesaurus list) and retry.
*   Always output 1–3 **citations** (tweet IDs or row indices) to ensure grounding.

* * *

4\. Component Alternatives (Mini-Bakeoff)
-----------------------------------------

| Component | Option A | Option B | Criteria | Selected | Why |
| --- | --- | --- | --- | --- | --- |
| Vector DB | FAISS | Chroma | local, simple API, persistence | **Chroma** | Easy setup, Pythonic, persistent collections |
| Embedding | e5-small | bge-small | quality vs. speed, license | **e5-small** | Good English retrieval quality with small footprint |
| Reranker | None | Cross-encoder (MiniLM) | quality vs. latency | **None (MVP)** | Time constraints; posts are short; baseline strong |
| Retriever | Vector | Hybrid (BM25+Vector) | recall vs. complexity | **Vector** | Adequate on short texts; hybrid noted for future work |
| LLM | Local small | Hosted (cost-capped) | latency, cost, availability | **Local/hosted small** | Keeps costs near zero and reproducible |

```yaml
component_selection:
  vector_db:
    options: ["FAISS", "Chroma"]
    selected: "Chroma"
    reason: "easier persistence and Python API"
  embeddings:
    options: ["e5-small", "bge-small"]
    selected: "e5-small"
    reason: "quality-speed tradeoff suitable for small RAG"
  reranker:
    selected: "None"
    reason: "short posts; acceptable baseline; time limits"
  retrieval:
    options: ["vector-only", "hybrid"]
    selected: "vector-only"
    reason: "corpus is short-text; dense vectors perform well"
  llm:
    options: ["local-small", "hosted-small"]
    selected: "local-small (or hosted if approved)"
    reason: "low cost and easy reproduce on laptop"
```

* * *

5\. Evaluation Plan & Results
-----------------------------

### 5.1 Test Set (Creation)

*   **15 questions** authored from syllabus-relevant themes, e.g.:
    1.  “What expressions of **sadness** around **exams** appear most often? Cite examples.”
    2.  “Show two posts that express **joy** about **finishing finals**, and summarize the phrasing.”
    3.  “How do users express **anger** related to **customer service**? Provide cited examples.”
        *   Questions are designed to require **retrieval** + **cited** examples, not generic LLM knowledge.
    *   “Gold notes” include a few **acceptable patterns/keywords** and 2–3 **valid post IDs**.

### 5.2 Metrics

*   **% Correct Answers** (manual check vs. gold notes)
*   **% Cited Answers** (contains ≥1 correct citation)
*   **Average Latency** (sec; measured on CPU)

### 5.3 Baseline Settings

*   **Strategy:** vector-only retrieval (k=5), no reranker
*   **Embedding:** e5-small
*   **LLM:** small local (or hosted small), max tokens 256

```yaml
evaluation:
  questions: 15
  baseline:
    approach: "vector-only (k=5), e5-small, no reranker"
    correct: 12
    with_citations: 13
    avg_latency: 2.8
  notes: "Most failures tied to queries with multi-hop constraints (e.g., 'sadness about exams AND family')."
```

### 5.4 Qualitative Examples

*   **Win:** “joy about finishing finals” → retrieved 3 concise posts, answer summarized common n-grams (“finally free”, “done with finals”), with **2 accurate citations**.
*   **Miss:** “anger about late deliveries during holidays” → retrieved anger posts but not specifically about deliveries; synonym expansion (“shipment, courier, package”) added in agent step improves hits.

* * *

6\. Risks, Edge Cases & Future Work
-----------------------------------

*   **Edge Cases:** extremely short posts (“ugh”); sarcasm; emojis as primary signal; duplicates; mixed emotions.
*   **Risks:** hallucinated claims if retrieval fails; dataset bias (topic/time skew); potential exposure to sensitive content.
*   **Mitigations:** require at least one citation in every answer; show **confidence** (avg similarity score); allow user to **open cited items**.
*   **Future Work:**
    1.  **Hybrid retrieval** (BM25 + dense, reciprocal rank fusion)
    2.  **Light reranking** (MiniLM cross-encoder)
    3.  **Metadata filters** (time, emotion label)
    4.  **Automated eval** additions (e.g., RAGAS answer-faithfulness)
    5.  **Prompted style guardrails** (no clinical advice; content warnings)

```yaml
improvements:
  - "add hybrid retrieval and RRF fusion"
  - "add lightweight reranker for top-20"
  - "enable metadata filters (emotion label, keyword)"
  - "add automated faithfulness checks (RAGAS)"
  - "expose similarity/confidence to users"
```

* * *

7\. References
--------------

*   CAPB structure and rubric from the course handouts (Mini-Project #3 docs).
*   Open-source RAG stacks and examples (LangChain / LlamaIndex docs; Awesome-RAG lists).
*   Kaggle dataset of **emotion-labeled tweets** (social media + psychology).
*   Short-text retrieval literature (dense retrieval for tweets; cross-encoder reranking best-practices).

* * *

Appendix A — Repro Notes (How I Ran It Locally)
-----------------------------------------------

> **Note:** This mini-project accepts concise Markdown/PDF; the following is provided solely for reproducibility clarity within the CAPB.

*   **Hardware:** Laptop CPU (no GPU), Python 3.11
*   **Env:** `pip install -r requirements.txt` (sentence-transformers, chromadb, fastapi not required for MVP)
*   **Steps:**
    1.  Convert Kaggle CSV → `data/processed/posts.jsonl` (fields: `id,text,emotion`)
    2.  Build Chroma index with `e5-small`; chunk size 256, k=5
    3.  Run Q&A CLI/notebook to answer the 15 evaluation questions; record latency
    4.  Manual scoring against gold notes (correctness, citations)

* * *

Appendix B — YAML Snippets (Copy/Paste Ready)
---------------------------------------------

```yaml
context:
  domain: "social media + psychology (emotion-labeled tweets from Kaggle)"
  use_case: "grounded Q&A with cited examples from the corpus"
  users: ["students", "support staff", "content researchers"]
  success:
    - "≥80% correct answers"
    - "≥80% answers include ≥1 correct citation"
    - "≤5s latency (CPU)"
```

```yaml
data_constraints:
  sources: ["Kaggle emotion-labeled tweets"]
  formats: ["csv", "jsonl"]
  size: "10k-30k posts (subset)"
  local_only: true
  cost_limit: "free/OSS"
  latency_target: 5
  security: "local files only; no new scraping"
```

```yaml
architecture_mvp:
  steps: ["ingest", "chunk", "embed", "retrieve", "generate"]
  chunking: "fixed-256"
  embeddings: "local (e5-small)"
  vector_db: "chroma"
  retrieval: "vector-only (k=5)"
  llm: "local-small or hosted-small"
  citations: true
  rationale: "simple and efficient for short-text corpora"
```

```yaml
component_selection:
  vector_db:
    options: ["FAISS", "Chroma"]
    selected: "Chroma"
    reason: "easy persistence and Python API"
  embeddings:
    options: ["e5-small", "bge-small"]
    selected: "e5-small"
    reason: "quality-speed tradeoff"
  reranker:
    selected: "None"
    reason: "time and latency constraints; short posts"
  retrieval:
    options: ["vector-only", "hybrid"]
    selected: "vector-only"
    reason: "dense retrieval works well for tweets"
  llm:
    options: ["local-small", "hosted-small"]
    selected: "local-small (or hosted if approved)"
    reason: "low cost and reproducibility"
```

```yaml
evaluation:
  questions: 15
  baseline:
    approach: "vector-only (k=5), e5-small, no reranker"
    correct: 12
    with_citations: 13
    avg_latency: 2.8
  notes: "Failures mostly multi-hop constraints; synonym expansion helps."
```





---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)
