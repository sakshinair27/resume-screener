# 📄 AI Resume Screener

> Upload resumes, paste a job description, get AI-ranked candidates with reasoning and source citations — powered by RAG.

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![LangChain](https://img.shields.io/badge/LangChain-0.3-1C3C3C?logo=langchain&logoColor=white)](https://www.langchain.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-gpt--4o--mini-412991?logo=openai&logoColor=white)](https://openai.com/)
[![ChromaDB](https://img.shields.io/badge/ChromaDB-0.5-FF6B6B)](https://www.trychroma.com/)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.40-FF4B4B?logo=streamlit&logoColor=white)](https://streamlit.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**🚀 Live demo:** _coming Day 5_ → will be `https://resume-screener-sakshi.streamlit.app`
**📺 Walkthrough video:** _coming Day 7_

---

## What it does

Recruiters often screen 100+ resumes per role using keyword matching, which misses strong candidates whose resumes phrase things differently than the job description. This tool uses **semantic search** (vector embeddings) and an LLM to rank candidates by *meaning*, not keywords — and shows its reasoning so users can verify the output.

**Demo flow:**
1. Upload PDF resumes through the web UI
2. Paste a job description
3. See candidates ranked by match score, with strengths, gaps, and the exact resume snippets that drove the score

![Screenshot placeholder — replace with real screenshot after Day 5](docs/screenshot.png)

---

## Why RAG?

A naive approach would stuff every resume into a single LLM prompt and ask for a ranking. That breaks at scale (context window limits, cost) and gives no traceability. **RAG (Retrieval-Augmented Generation)** solves both:

- **Retrieval** narrows N resumes down to the top-k relevant chunks per query, keeping prompts small and cheap
- **Generation** scores only those candidates, with reasoning grounded in the retrieved evidence
- **Citations** — every score links back to the source chunks, so users can verify the LLM isn't hallucinating

---

## Architecture

```
┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│   Streamlit UI   │      │  Ingestion (one  │      │ Vector Store     │
│                  │      │  time per resume)│      │  (ChromaDB)      │
│  • Upload PDFs   │─────►│  PyPDFLoader →   │─────►│  • 1536-dim vecs │
│  • Paste JD      │      │  Splitter →      │      │  • metadata:     │
│  • View results  │      │  OpenAI Embed    │      │    candidate,    │
└────────┬─────────┘      └──────────────────┘      │    chunk_id      │
         │                                          └────────┬─────────┘
         │  job description                                  │
         ▼                                                   │
┌──────────────────┐                                         │
│  Retrieval       │                                         │
│  • semantic      │◄────────────────────────────────────────┘
│    search top-k  │           top-k chunks per candidate
│  • per-candidate │
│    grouping      │
└────────┬─────────┘
         │  candidate evidence
         ▼
┌──────────────────┐      ┌──────────────────┐
│  LLM Scoring     │      │  Pydantic Schema │
│  (gpt-4o-mini)   │─────►│  • match_score   │
│  • structured    │      │  • strengths[]   │
│    output        │      │  • gaps[]        │
│  • temp=0        │      │  • verdict       │
└────────┬─────────┘      └──────────────────┘
         │
         ▼
   Ranked results
   with citations
```

---

## Tech stack & key decisions

| Component | Choice | Why |
|-----------|--------|-----|
| **LLM** | OpenAI `gpt-4o-mini` | Cheapest capable model; ~$0.15/1M input tokens. Total dev cost: ~$2. |
| **Embeddings** | OpenAI `text-embedding-3-small` | 1536-dim, $0.02/1M tokens, strong on short documents like resumes |
| **Vector DB** | ChromaDB (local) | File-based, zero setup, no cloud account. Trade-off: not distributed (fine for ≤10K resumes). |
| **Orchestration** | LangChain LCEL | Modern pipe-syntax chains; clean separation of retrieval and generation |
| **PDF parsing** | `pypdf` | Maintained successor to PyPDF2 (deprecated 2022) |
| **UI** | Streamlit | Pure Python, free hosting via Streamlit Community Cloud |
| **Structured output** | Pydantic + `with_structured_output()` | Forces JSON conformance; no regex parsing |

---

## Quickstart

```bash
# 1. Clone and enter
git clone https://github.com/<your-username>/resume-screener.git
cd resume-screener

# 2. Create environment
python3.10 -m venv .venv
source .venv/bin/activate         # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 3. Add your OpenAI key
cp .env.example .env              # edit .env, paste key

# 4. Generate sample PDFs (3 fake resumes for testing)
pip install reportlab
python src/make_sample_pdfs.py

# 5. Run the app
streamlit run src/app.py
```

Open http://localhost:8501.

Full setup details, troubleshooting, and the day-by-day build log are in [`docs/BUILD_LOG.md`](docs/BUILD_LOG.md).

---

## Project structure

```
resume-screener/
├── src/
│   ├── app.py                   # Streamlit UI
│   ├── ingest.py                # PDF → chunks → embeddings → ChromaDB
│   ├── screener.py              # Retrieval + LLM scoring chain
│   ├── schemas.py               # Pydantic models for structured output
│   └── make_sample_pdfs.py      # Test data generator
├── sample_data/                 # Sample resumes + job description (text)
├── data/
│   ├── resumes/                 # User-uploaded PDFs (gitignored)
│   └── chroma_db/               # Vector store (gitignored)
├── notebooks/
│   └── evaluation.ipynb         # Retrieval + ranking accuracy metrics
├── docs/
│   ├── BUILD_LOG.md             # Day-by-day notes
│   └── screenshot.png
├── tests/
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Evaluation

To validate the system isn't just "vibes," I built a small evaluation harness:

- **Dataset:** 10 hand-labeled resumes scored 1-5 against a sample job description
- **Retrieval metric:** Recall@5 — does the top-5 retrieval include the chunks that matter?
- **Ranking metric:** Spearman correlation between LLM ranking and human ranking
- **Cost metric:** Average tokens + dollars per screening run

_Results table coming Day 6._ Full methodology in [`notebooks/evaluation.ipynb`](notebooks/evaluation.ipynb).

---

## What I learned

- **LCEL is worth the learning curve.** The `prompt | llm | parser` pipe syntax is much cleaner than nested chains, but tutorials are inconsistent — I had to read the LangChain source to understand `RunnablePassthrough` and parallel branches.
- **Chunk size matters more than I expected.** Resumes are short (1-2 pages); `chunk_size=1000` often produced just 1-2 chunks per resume, which hurt per-section retrieval. I experimented with section-aware splitting (Experience, Skills, Education) and got better results.
- **Structured output >> string parsing.** `with_structured_output()` against Pydantic schemas eliminated an entire class of bugs (LLM returning malformed JSON, missing fields).
- **RAG is mostly retrieval, not generation.** When results felt off, 9 times out of 10 the fix was a better chunking strategy or metadata filter, not a better prompt.

---

## Roadmap

- [ ] **Hybrid search** — combine semantic (ChromaDB) with keyword (BM25) for better recall on technical terms
- [ ] **Reranking** — Cohere Rerank as a second-stage filter to boost precision on the top-N
- [ ] **Multi-tenant auth** — let multiple recruiters maintain separate resume pools
- [ ] **Bias auditing** — measure score variance across demographic-correlated variations of the same resume
- [ ] **Async ingestion** — current pipeline blocks on each PDF; batch + parallel would help for 100+ resumes
- [ ] **PostgreSQL + pgvector** — for production scale beyond ChromaDB's local-file model

---

## License

MIT — see [LICENSE](LICENSE).

---

## About

Built by **Sakshi Nair** as a personal project to learn RAG architecture hands-on and explore production patterns for LLM applications — semantic retrieval, structured output, and evaluation.

I'm an MS Data Science graduate from **Indiana University** (2026, GPA 3.9) actively looking for **Data Scientist / ML Engineer** roles.

🔗 [LinkedIn](https://www.linkedin.com/in/sakshinair27/) · ✉️ sakshinair086@gmail.com · 💻 [Other projects](https://github.com/sakshinair27)
