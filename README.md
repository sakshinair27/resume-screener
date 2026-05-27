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
