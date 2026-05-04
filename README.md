# -Murabaha-AI-Assistant

# ☽ Murabaha AI Assistant

> A Shari'a-compliant Islamic finance chatbot for **Buraq Bank Pakistan**, powered by AAOIFI Shari'a Standard No. 08 (Murabaha). Built with a RAG (Retrieval-Augmented Generation) pipeline using Python, FastAPI, ChromaDB, and React.

---

## 📸 Preview

![Murabaha AI Assistant UI](./docs/preview.png)

---

## 🌟 Features

- **Clause-cited answers** — every response references the exact SS-08 clause (e.g. "Per clause 2/3/1...")
- **RAG pipeline** — retrieves relevant clauses from ChromaDB before asking GPT, no hallucination
- **Session memory** — follow-up questions are rewritten into standalone queries before retrieval
- **Topic browser sidebar** — all 10 SS-08 sections browsable (§1 Scope through §5 Receivables)
- **Compliance Mode** — toggle to enforce strict clause citation in every answer
- **Two-gate topic filter** — keyword check + similarity score rejects off-topic questions before LLM call
- **Answer cache** — repeated questions served instantly at zero cost
- **Citation validation** — hallucinated clause numbers flagged automatically
- **Glossary panel** — Islamic finance terms (Hamish Jiddiyyah, Arboun, Inah, Takaful, etc.)
- **Islamic aesthetic UI** — deep green sidebar, gold accents, crescent moon motif, Urdu greeting
- **Mobile responsive** — sliding sidebar, works on all screen sizes

---

## 🏗️ Architecture

```
User Question
      │
      ▼
┌─────────────────┐
│  React Frontend │  localhost:3000
│  (MurabahaChat) │
└────────┬────────┘
         │ POST /chat/session
         ▼
┌─────────────────┐
│  FastAPI Backend│  localhost:8000
│   (api.py)      │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────┐
│           rag.py (RAG Engine)       │
│                                     │
│  1. Two-gate topic filter           │
│  2. Cache check                     │
│  3. similarity_search (ChromaDB)    │
│  4. Score threshold check           │
│  5. Build prompt with clauses       │
│  6. GPT-4o-mini generates answer    │
│  7. Validate cited clauses          │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────┐
│   ChromaDB      │  ./chroma_db/
│  67 SS-08 chunks│
│  + embeddings   │
└─────────────────┘
```

---

## 📁 Project Structure

```
murabaha_bot/
├── 📄 SS-08 Murabaha.pdf          # Source document (AAOIFI Shari'a Standard)
├── 📄 murabaha_clean.txt          # Cleaned extracted text
├── 📄 murabaha_qa_pairs.json      # 55 Q&A test pairs for evaluation
├── 🐍 text_extraction.py          # PDF → clean text pipeline
├── 🐍 chunk_and_ingest.py         # Chunking + embedding + ChromaDB ingestion
├── 🐍 rag.py                      # RAG engine (ask, cache, filter, validate)
├── 🐍 api.py                      # FastAPI backend server
├── 🗄️  chroma_db/                 # Vector database (67 clause chunks)
├── 📄 .env                        # API keys (not committed)
├── 📄 requirements.txt            # Python dependencies
│
└── murabaha-frontend/             # React frontend
    ├── 📄 index.html
    ├── 📄 package.json
    ├── 📄 vite.config.js
    └── src/
        ├── 📄 main.jsx
        └── 📄 MurabahaChat.jsx    # Complete React app
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

Create a `.env` file in the root:

```env
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

### 4. Build the knowledge base (run once)

```bash
# Extract and clean the PDF
python text_extraction.py

# Chunk, embed and store in ChromaDB
python chunk_and_ingest.py
```

Expected output:
```
Extracted 67 clause-level chunks
Built 67 LangChain documents with metadata
Embedding and storing in ChromaDB...
Done. 67 clauses stored in ./chroma_db/
```

### 5. Test the RAG pipeline

```bash
# Quick 5-question test
python rag.py

# Full 55-question test suite
python rag.py --full
```

Target score: **92%+** (strict clause citation mode)

### 6. Start the backend API

```bash
uvicorn api:app --reload --port 8000
```

API docs available at: `http://localhost:8000/docs`

### 7. Start the React frontend

```bash
cd murabaha-frontend
npm install
npm run dev
```

Open: `http://localhost:3000`

---

## 🧠 RAG Pipeline Details

The RAG engine (`rag.py`) processes every question through 7 steps:

1. **Gate 1 — Keyword filter** (free, instant): checks if question contains Murabaha-related keywords
2. **Cache check**: returns cached answer instantly if question was asked before
3. **Single retrieval**: `similarity_search_with_relevance_scores(question, k=6)` against ChromaDB
4. **Gate 2 — Score threshold**: if top score < 0.05, returns "not found" without calling LLM
5. **Topic filter**: if no keywords AND low score → rejects as off-topic
6. **LLM call**: GPT-4o-mini receives retrieved clause text + strict Shari'a prompt
7. **Citation validation**: regex checks answer for hallucinated clause numbers

### Threshold Configuration

```python
SIMILARITY_THRESHOLD  = 0.05   # Below this → not_found (no LLM call)
TOPIC_SCORE_THRESHOLD = 0.00   # Below this AND no keywords → off_topic
CONFIDENCE_THRESHOLD  = 0.15   # Below this → low_confidence flag
```

---

## 🌐 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Server health check |
| `POST` | `/chat` | Single question, stateless, cached |
| `POST` | `/chat/session` | Conversation with memory |
| `GET` | `/session/{id}` | Get conversation history |
| `DELETE` | `/session/{id}` | Clear session memory |

### Example request

```bash
curl -X POST http://localhost:8000/chat/session \
  -H "Content-Type: application/json" \
  -d '{"question": "Is bilateral promise allowed in Murabaha?", "session_id": "user-123"}'
```

### Example response

```json
{
  "question": "Is bilateral promise allowed in Murabaha?",
  "answer": "Per clause 2/3/1, a bilateral promise binding on both parties is NOT PERMISSIBLE. However, per clause 2/3/3, it is permissible if there is an option to cancel exercisable by either party.",
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
Results: 53 passed / 2 failed / 55 total
Score:   96%  (strict citation rule)
```

Failures are edge cases where document wording differs significantly from question phrasing. The RAG engine itself is working correctly for all 51 passing questions.

---

## 📚 Knowledge Base

Built from **AAOIFI Shari'a Standard No. 08 — Murabaha**, covering:

| Section | Topic | Clauses |
|---|---|---|
| §1 | Scope of the Standard | 1 |
| §2 | Procedures Prior to Contract | 2/1 – 2/5 |
| §3 | Asset Acquisition & Possession | 3/1 – 3/2 |
| §4 | Conclusion of Murabaha Contract | 4/1 – 4/11 |
| §5 | Guarantees & Receivables | 5/1 – 5/10 |

---

## 🔐 Environment Variables

| Variable | Description |
|---|---|
| `OPENAI_API_KEY` | Your OpenAI API key (required) |

---

## 📦 Python Dependencies

```txt
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

Install all:
```bash
pip install -r requirements.txt
```

---

## ⚠️ Disclaimer

This chatbot provides guidance based on AAOIFI Shari'a Standard No. 08 only. For binding legal opinions or complex transactions, always consult your **Shari'a Supervisory Board**. Answers are indicative and not a substitute for qualified Shari'a advice.

---

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 👤 Author
humaila0

<div align="center">
  <sub>Built with ☽ for Islamic Finance · Powered by AAOIFI SS-08</sub>
</div>
