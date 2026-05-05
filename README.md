# ☽ Murabaha AI Assistant

> A Shari'a-compliant Islamic finance chatbot for **Buraq Bank Pakistan**, powered by AAOIFI Shari'a Standard No. 08 (Murabaha). Built with a RAG pipeline using Python, FastAPI, ChromaDB, and React.

---

## 📸 Preview

![Murabaha AI Assistant](/preview.png)

---

## 🌟 Features

- **Two answering modes** — Compliance Mode (strict clause-only) and Learning Mode (friendly explanations)
- **Clause-cited answers** — every response references the exact SS-08 clause (e.g. "Per clause 2/3/1...")
- **Query routing** — smart keyword-based routing directs questions to the most relevant clause groups before vector search
- **Hard-coded answers** — general questions like "Is Murabaha halal?" and "What is Murabaha?" return instant accurate answers
- **RAG pipeline** — retrieves top-8 relevant clauses from ChromaDB before asking GPT
- **Session memory** — follow-up questions are rewritten into standalone queries before retrieval
- **Topic browser sidebar** — all 10 SS-08 sections (§1 through §5/10) clickable
- **Two-gate topic filter** — keyword check + similarity score rejects off-topic questions
- **Answer cache** — per-mode caching (compliance and learning cached separately)
- **Citation validation** — hallucinated clause numbers flagged automatically
- **Glossary panel** — 10 Islamic finance terms with definitions
- **Islamic aesthetic UI** — deep green sidebar, gold accents, crescent moon, Urdu greeting
- **Mobile responsive** — sliding sidebar, works on all screen sizes

---

## 🏗️ Architecture

```
User Question
      │
      ▼
┌─────────────────┐
│  React Frontend │  localhost:3000
│  MurabahaChat   │  Compliance / Learning mode toggle
└────────┬────────┘
         │ POST /chat/session  (with mode: "compliance"|"learning")
         ▼
┌─────────────────┐
│  FastAPI Backend│  localhost:8000
│   (api.py)      │
└────────┬────────┘
         │
         ▼
┌──────────────────────────────────────────────┐
│              rag.py (RAG Engine)             │
│                                              │
│  1. Hard-code check (Murabaha definition,    │
│     halal questions → instant answer)        │
│  2. Query routing (_route_question)          │
│     → maps question to best clause keywords  │
│  3. Cache check (per mode)                   │
│  4. similarity_search_with_relevance_scores  │
│     k=8 for broader retrieval                │
│  5. Two-gate topic filter                    │
│  6. Score threshold check                    │
│  7. Choose prompt by mode                    │
│     (compliance or learning prompt)          │
│  8. GPT-4o-mini generates answer             │
│  9. Validate cited clauses                   │
└────────┬─────────────────────────────────────┘
         │
         ▼
┌─────────────────┐
│   ChromaDB      │  ./chroma_db/
│  67 SS-08 chunks│  + optional definition docs
└─────────────────┘
```

---

## 📁 Project Structure

```
murabaha_bot/
├── 📄 SS-08 Murabaha.pdf           # Source — AAOIFI Shari'a Standard No. 08
├── 📄 murabaha_clean.txt           # Cleaned extracted text
├── 📄 murabaha_qa_pairs.json       # 55 Q&A test pairs
├── 🐍 text_extraction.py           # PDF → clean text
├── 🐍 chunk_and_ingest.py          # Chunk + embed + store in ChromaDB
├── 🐍 rag.py                       # RAG engine with modes + routing
├── 🐍 api.py                       # FastAPI backend
├── 🐍 add_definition.py            # (optional) adds general Murabaha definition chunk
├── 🗄️  chroma_db/                  # Vector database
├── 📄 .env                         # API keys (not committed)
├── 📄 requirements.txt             # Python dependencies
│
└── murabaha-frontend/              # React frontend
    ├── 📄 index.html
    ├── 📄 package.json
    ├── 📄 vite.config.js
    └── src/
        ├── 📄 main.jsx
        └── 📄 MurabahaChat.jsx     # Complete React app
```

---

## ⚙️ Tech Stack

| Layer | Technology |
|---|---|
| LLM | GPT-4o-mini (OpenAI) |
| Embeddings | text-embedding-3-small (OpenAI) |
| Vector DB | ChromaDB |
| RAG Framework | LangChain |
| Backend | FastAPI + Uvicorn |
| Frontend | React 18 + Vite |
| Knowledge Base | AAOIFI SS-08 (67 clause chunks) |

---

## 🚀 Getting Started

### Prerequisites
- Python 3.10+
- Node.js 18+
- OpenAI API key

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/murabaha-ai-assistant.git
cd murabaha-ai-assistant
```

### 2. Set up Python environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Mac/Linux
source venv/bin/activate

pip install -r requirements.txt
```

### 3. Add your OpenAI API key

```env
# .env
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

### 4. Build the knowledge base (once only)

```bash
python text_extraction.py   # Clean the PDF
python chunk_and_ingest.py  # Build ChromaDB
```

### 5. (Optional) Add general Murabaha definition

```bash
python add_definition.py    # Adds definition + process chunks for broad questions
```

### 6. Test RAG pipeline

```bash
python rag.py          # Quick 5-question test
python rag.py --full   # Full 55-question suite (target: 92%+)
```

### 7. Start backend

```bash
uvicorn api:app --reload --port 8000
```

### 8. Start frontend

```bash
cd murabaha-frontend
npm install
npm run dev
```

Open: **http://localhost:3000**

---

## 🧠 RAG Pipeline — Detailed Flow

### Query Routing

Before vector search, `_route_question()` maps common question patterns to the most relevant clause keywords. This dramatically improves retrieval for known question types:

```python
# Examples
"hamish jiddiyyah" → "Clause 2/5/3 2/5/4 2/5/5 Hamish Jiddiyyah security deposit..."
"bilateral promise" → "Clause 2/3/1 2/3/3 bilateral promise binding Murabaha"
"bill of lading"   → "Clause 3/2/4 bill of lading constructive possession"
"late payment"     → "Clause 4/8 delay payment extra payment late installment"
```

### Hard-Coded Answers

General questions that don't match specific clauses are handled with pre-written answers:

```python
# "is murabaha halal" → instant answer citing 3/1/1 and 4/6
# "what is murabaha"  → definition + process explanation
```

### Answering Modes

```python
# Compliance prompt — strict clause-only, always cites, states NOT PERMISSIBLE
# Learning prompt   — friendly, full summary, extra detail, simple language
```

### Threshold Configuration

```python
SIMILARITY_THRESHOLD  = 0.03   # Below this → not_found
TOPIC_SCORE_THRESHOLD = 0.00   # Below this AND no keywords → off_topic
CONFIDENCE_THRESHOLD  = 0.12   # Below this → low_confidence flag
```

---

## 🌐 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Server health check |
| `POST` | `/chat` | Single question, stateless, cached |
| `POST` | `/chat/session` | Conversation with memory |
| `GET` | `/session/{id}` | Get conversation history |
| `DELETE` | `/session/{id}` | Clear session |

### Request with mode

```json
{
  "question": "Is bilateral promise allowed in Murabaha?",
  "session_id": "user-123",
  "mode": "compliance"
}
```

`mode` options: `"compliance"` (default) or `"learning"`

### Example response

```json
{
  "question": "Is bilateral promise allowed in Murabaha?",
  "answer": "Per clause 2/3/1, NOT PERMISSIBLE. However, per clause 2/3/3, permissible if cancellation option exists.",
  "retrieved_clauses": ["2/3/1", "2/3/2", "2/3/3"],
  "cited_clauses": ["2/3/1", "2/3/3"],
  "invalid_citations": [],
  "status": "answered",
  "source": "llm",
  "low_confidence": false,
  "top_score": 0.635
}
```

---

## 📊 Test Suite Results

```
Results: 51 passed / 4 failed / 55 total
Score:   92.7%  (strict citation rule, both clause-in-answer AND status != not_found required)
```

---

## 📚 SS-08 Coverage

| Section | Topic | Key Clauses |
|---|---|---|
| §1 | Scope | General definition |
| §2/1–2/5 | Pre-Contract Procedures | Promise, Hamish Jiddiyyah, Arboun, fees |
| §3/1–3/2 | Asset Acquisition & Possession | Ownership, bill of lading, insurance |
| §4/1–4/11 | Contract Conclusion | Pricing, profit, defects, disclosure |
| §5/1–5/10 | Guarantees & Receivables | Late payment, rescheduling, early settlement |

---

## 🔐 Environment Variables

| Variable | Description |
|---|---|
| `OPENAI_API_KEY` | OpenAI API key (required) |

---

## 📦 Python Dependencies

```
langchain
langchain-openai
langchain-chroma
langchain-community
chromadb
fastapi
uvicorn
python-dotenv
pypdf
```

---

## ⚠️ Disclaimer

This chatbot provides guidance based on AAOIFI Shari'a Standard No. 08 only. For binding opinions or complex transactions, always consult your **Shari'a Supervisory Board**.

---

## 👤 Author

humaila0

---

<div align="center">
  <sub>Built with ☽ for Islamic Finance · AAOIFI SS-08 · Buraq Bank Pakistan</sub>
</div>
