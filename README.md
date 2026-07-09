**TaxShield** is an AI-powered legal automation platform designed to help Chartered Accountants (CAs) and tax practitioners handle Indian GST (Goods & Services Tax) notices efficiently. 

By ingesting scanned or digital GST notices (PDFs), TaxShield runs them through a sophisticated **5-agent LangGraph pipeline** to perform Optical Character Recognition (OCR), entity extraction, risk classification, legal analysis, and draft generation—all while strictly adhering to data privacy standards (DPDP Act, 2023) by running PII redaction locally.

## ✨ Features

- 📄 **Smart Document Processing**: Converts both digital and scanned PDFs to text using PyMuPDF and Surya OCR (runs locally).
- 🔒 **Privacy First (DPDP Compliant)**: Redacts PII (PAN, Aadhaar, phone numbers) using Microsoft Presidio locally *before* any data reaches external LLMs. Data is auto-deleted after 90 days.
- ⚖️ **Legal Analysis & Defense Drafting**: Uses Groq (Llama 3.3 70B) and Gemini models to determine if a notice is time-barred and drafts a substantive legal response based on a curated Knowledge Base of CGST Sections and CBIC circulars.
- 🛡️ **Hallucination Mitigation (InEx)**: A 5-stage verification pipeline (Citation Grounding, Multi-Sample Consistency, CoVe, Self-Consistency, and Adversarial Challenge) ensures the drafted response relies on actual legal precedent, not LLM hallucinations.
- 🔍 **Hybrid RAG Search**: Uses a combination of BM25 (keyword) and Supabase pgvector (semantic) search with Reciprocal Rank Fusion (RRF) to retrieve the most relevant CBIC circulars.
- 🖥️ **Modern Web Dashboard**: A Next.js 16 (App Router) interface for CAs to review the extracted data, assess risk levels, and edit the final drafted reply.

## 🏗️ Architecture & Tech Stack

### Frontend
* **Framework**: Next.js 16 (React 19, TypeScript)
* **Styling**: TailwindCSS v4, Lucide React

### Backend
* **Framework**: FastAPI 2.0 (Python 3.11), Uvicorn
* **Database**: Supabase PostgreSQL (Prod) / SQLite (Dev) via SQLAlchemy (Async)
* **Authentication**: JWT, bcrypt
* **Rate Limiting**: Redis

### AI & Agents
* **Orchestration**: LangGraph, LangChain
* **LLMs**: Groq (Llama 3.3 / 3.1), Google Gemini (2.0 Flash / 1.5 Pro)
* **Embeddings**: OpenAI (`text-embedding-3-small`)
* **Vector Store**: Supabase `pgvector`
* **Local NLP**: Surya OCR, PyMuPDF, Presidio (spaCy)

---

## 🧠 How the AI Pipeline Works

TaxShield uses a **LangGraph StateGraph** to orchestrate 5 specialized agents:

1. **Agent 1 (Document Processor)**: Handles OCR, sanitizes prompt injections, redacts PII, extracts entities (GSTIN, DIN, Sections, Amounts), and runs a preliminary time-bar calculation.
2. **Agent 2 (Risk Router)**: Makes the final time-bar decision based on the specific CGST section and classifies the risk level (LOW, MEDIUM, HIGH).
3. **Agent 3 (Legal Analyst)**: Searches the knowledge base for circulars, detects contradictions in the notice, and builds a JSON defense strategy.
4. **Agent 4 (Master Drafter)**: Takes the defense strategy and writes a professional, legally grounded draft reply.
5. **Agent 5 (InEx Verifier)**: A 5-stage hallucination check to ensure citations are grounded and logic is consistent before presenting the draft to the user.



## 📚 Knowledge Base
TaxShield relies on a meticulously hand-curated knowledge base of Markdown files (`backend/data/curated_kb/`) covering CGST Sections (e.g., 73, 74, 16, 50) and CBIC Circulars. A weekly background scraper ensures new circulars are fetched from the CBIC website and staged for CA review before being added to the active RAG store.
