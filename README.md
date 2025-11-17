# 🧠 Mini Project #3 — Agentic RAG over Social-Media Emotion Posts  
*AI Mini-Project Series | Composable AI Project Blueprint (CAPB)*

## 📘 Overview
This repository contains my submission for **Mini-Project #3** in the *AI Mini-Project Series*.  
The goal is to design and document a **Retrieval-Augmented Generation (RAG)** pipeline using the **CAPB (Composable AI Project Blueprint)** framework.

My chosen domain is **social media and psychology** — specifically, understanding how emotions are expressed across posts.  
Using the **Kaggle Emotion-Labeled Tweets** dataset, this system performs **grounded question-answering**.
---

## 🎯 Project Objectives
- Build a **minimal viable RAG system (MVP)** for short-text corpora.  
- Evaluate **retrieval quality, grounding accuracy, and latency**.  
- Demonstrate clear **design reasoning and reproducibility** through CAPB documentation.  
- Explore how RAG can support **trustworthy and explainable insights** from public online text.

---

## 🧩 CAPB Structure
This project follows the official CAPB format:

| Section | Description |
|----------|-------------|
| **1. Project Context** | Defines domain, use case, users, and measurable success metrics. |
| **2. Data & Constraints** | Describes dataset sources, preprocessing, and ethical limitations. |
| **3. RAG Architecture (MVP)** | Outlines the pipeline, components, and rationale for design choices. |
| **4. Component Bakeoff** | Compares tool alternatives (e.g., Chroma vs. FAISS, e5-small vs. bge-small). |
| **5. Evaluation** | Provides 15 manual Q&A tests with retrieval metrics and latency results. |
| **6. Risks & Future Work** | Notes hallucination risks, data bias, and next steps for improvement. |
| **7. References** | Lists supporting resources (papers, repos, datasets). |

---

## 💾 Dataset
**Source:** [Kaggle – Emotion-Labeled Tweets Dataset](https://www.kaggle.com/datasets)  
**Domain:** Social Media / Psychology  
**Description:** Each record contains a short tweet and an associated emotion label (*joy, sadness, anger, fear, love, surprise*).  
**Subset Used:** ~10k–30k posts, cleaned and anonymized.  
**Format:** `CSV → JSONL` with fields `{id, text, emotion}`

**Ethical Use:**  
All data are public and used only for educational demonstration.  
No scraping or PII beyond provided text.  
No claims of psychological assessment are made.


