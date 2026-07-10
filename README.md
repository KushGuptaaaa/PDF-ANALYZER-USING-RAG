# 📚 RAG Book Assistant

A Retrieval-Augmented Generation (RAG) application that lets you upload a PDF and ask natural-language questions about its content. Instead of relying on an LLM's memory or hallucinating answers, the app retrieves the most relevant chunks of the actual document and grounds every answer strictly in that context.

---

## 🚀 Features

- **PDF Upload & Parsing** — Upload any PDF book/document directly through the UI
- **Automatic Chunking** — Splits large documents into overlapping, semantically coherent chunks for better retrieval
- **Vector Embeddings** — Converts text chunks into embeddings using OpenAI's embedding models
- **Persistent Vector Store** — Stores embeddings locally using ChromaDB, so you don't need to re-process a PDF every session
- **MMR-based Retrieval** — Uses Maximal Marginal Relevance search to fetch relevant *and* diverse chunks, reducing redundant context
- **Grounded Answers Only** — The LLM is explicitly prompted to answer only from retrieved context, and to say so honestly if the answer isn't in the document
- **Simple Web UI** — Built with Streamlit for quick interaction, no separate frontend needed

---

## 🧠 How It Works

```
PDF Upload
    ↓
Text Extraction (PyPDFLoader)
    ↓
Chunking (RecursiveCharacterTextSplitter)
    ↓
Embedding (OpenAI Embeddings)
    ↓
Vector Store (ChromaDB, persisted to disk)
    ↓
User Question → Retrieve Top-k Relevant Chunks (MMR)
    ↓
Context + Question → Prompt Template → LLM (Mistral)
    ↓
Grounded Answer
```

### Why RAG instead of just asking an LLM directly?

Feeding an entire book into an LLM either exceeds its context window or drowns the model in irrelevant text, often leading to less accurate answers (the "lost in the middle" problem). RAG solves this by retrieving only the handful of chunks actually relevant to the question, then asking the LLM to answer using just that focused context — improving both accuracy and cost efficiency.

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| PDF Loading | `PyPDFLoader` (LangChain Community) |
| Text Splitting | `RecursiveCharacterTextSplitter` |
| Embeddings | `OpenAIEmbeddings` |
| Vector Database | `ChromaDB` |
| LLM | `ChatMistralAI` (mistral-small) |
| Orchestration | `LangChain` |
| Frontend | `Streamlit` |
| Environment Config | `python-dotenv` |

---

## 📂 Project Structure

```
├── ingest.py          # Standalone script: load, chunk, embed, and store a PDF
├── app.py             # Streamlit app: upload PDF, build vector DB, ask questions
├── chroma_db/         # Persisted vector store (created after first run)
├── .env               # API keys (not committed)
└── README.md
```

---

## ⚙️ Setup & Installation

### 1. Clone the repository
```bash
git clone <your-repo-url>
cd rag-book-assistant
```

### 2. Create a virtual environment
```bash
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install streamlit langchain langchain-community langchain-openai langchain-mistralai chromadb pypdf python-dotenv
```

### 4. Set up environment variables
Create a `.env` file in the project root:
```
OPENAI_API_KEY=your_openai_api_key
MISTRAL_API_KEY=your_mistral_api_key
```

### 5. Run the app
```bash
streamlit run app.py
```

The app will open at `http://localhost:8501`.

---

## 💡 Usage

1. Upload a PDF using the file uploader
2. Click **"Create Vector Database"** to chunk, embed, and store the document
3. Once processing completes, type your question in the input box
4. The app retrieves the most relevant chunks and returns an answer grounded in the document — or honestly tells you if the answer isn't present

---

## 🔍 Key Design Choices

- **Chunk size (1000) with overlap (200)** — Balances retrieval granularity with preserving context across chunk boundaries, preventing answers from being cut off mid-sentence.
- **MMR retrieval (`fetch_k=10`, `k=4`, `lambda_mult=0.5`)** — Rather than naively grabbing the top-k most similar chunks (which can be redundant), MMR balances relevance with diversity, so retrieved chunks cover different parts of the answer instead of repeating the same section.
- **Strict grounding prompt** — The system prompt explicitly instructs the model to answer *only* from provided context and to admit when it can't find an answer, reducing hallucination risk.
- **Persisted vector store** — Embeddings are computed once and reused across sessions, avoiding redundant API calls and cost for the same document.

---

## 🚧 Known Limitations / Future Improvements

- No OCR support — scanned/image-based PDFs won't extract text correctly
- Single-document only — no multi-PDF search or cross-document Q&A yet
- No source citation — answers don't currently indicate which page/section they came from
- No chat history — each question is independent, with no conversational memory
- Vector store isn't namespaced per document — uploading a new PDF currently reuses/overwrites the same `chroma_db` directory

---

## 📄 License

MIT License — feel free to use and modify.