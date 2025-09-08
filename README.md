# 🧠 Chatbot Using RAG and LangChain

A Streamlit-based chatbot powered by Retrieval-Augmented Generation (RAG) and OpenAI. Upload your PDFs and chat with them! This app leverages LangChain, FAISS, and OpenAI’s GPT models to extract and query document content with metadata-aware answers.

![App Screenshot](thumbnail.webp)

---

## 🔧 Features

* 🔍 **Upload multiple PDFs** and query across all of them
* 📄 **Metadata-rich answers** with filename and page references
* 🧠 Uses **LangChain + FAISS** for semantic search
* 🤖 **Streamlit Chat UI** for natural conversation
* 💾 **OpenAI API support** with streaming responses

---

## 📁 Project Structure

```
.
├── .gitignore
├── LICENSE
├── README.md             # ← You're reading it
├── app.py                # Main Streamlit app
├── brain.py              # PDF parsing and vector index logic
├── compare medium.gif    # Optional UI illustration
├── requirements.txt      # Python dependencies
└── thumbnail.webp        # Preview image
```

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/shamazhooda/chatbot-rag-langchain.git
cd chatbot-rag-langchain
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Set OpenAI API Key

Create a `.streamlit/secrets.toml` file with:

```toml
OPENAI_API_KEY = "your-openai-key"
```

Or export it via environment variable:

```bash
export OPENAI_API_KEY="your-openai-key"
```

### 4. Run the App

```bash
streamlit run app.py
```

---

## 📚 How It Works

1. **Upload PDFs** via the UI
2. Each PDF is parsed using `PyPDF2` and chunked via LangChain’s `RecursiveCharacterTextSplitter`
3. Chunks are embedded using OpenAI Embeddings
4. Stored in a FAISS vector store for semantic similarity search
5. Queries are matched to top PDF chunks and passed to ChatGPT with context
6. Answers include **file name** and **page number** metadata for citation

---

## 🛠️ Tech Stack

* [Streamlit](https://streamlit.io/) – UI framework
* [LangChain](https://www.langchain.com/) – PDF chunking and retrieval
* [FAISS](https://github.com/facebookresearch/faiss) – Vector search backend
* [OpenAI GPT](https://platform.openai.com/docs) – LLM-based answer generation
* [PyPDF2](https://pypi.org/project/pypdf/) – PDF parsing

---

## ✅ Example Prompt

> "What are the main points from the introduction?"

> Answer: The introduction highlights... **(example.pdf, page 1)**

---

## Architecture and Storage

- UI: `app.py` (Streamlit)
- RAG pipeline: `brain.py` (parse → chunk → embed → index)
- Model: OpenAI Chat Completions API (streaming)
- Retriever: FAISS similarity search

### Storage
- PDFs: in-memory during session (not persisted by default).
- Chunks/metadata: in-memory `Document` objects.
- Vector store: FAISS, created in-memory and cached via `st.cache_resource`. Lives for the server process lifetime; cleared on restart or cache clear.
- Chat history: `st.session_state` (per browser session).

### Optional persistence
- To keep vectors across restarts, persist FAISS:

```python
# brain.py
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import FAISS

def save_index_local(vectordb: FAISS, path: str):
    vectordb.save_local(path)

def load_index_local(path: str, openai_api_key: str) -> FAISS:
    embeddings = OpenAIEmbeddings(openai_api_key=openai_api_key)
    return FAISS.load_local(path, embeddings, allow_dangerous_deserialization=True)
```

```python
# app.py (around create_vectordb)
DB_DIR = "data/faiss_index"
if os.path.isdir(DB_DIR):
    vectordb = load_index_local(DB_DIR, OPENAI_API_KEY)
else:
    vectordb = get_index_for_pdf([f.getvalue() for f in files], filenames, OPENAI_API_KEY)
    save_index_local(vectordb, DB_DIR)
```

### What each module does

- `app.py`
  - Loads `OPENAI_API_KEY` from `st.secrets` or environment.
  - Uploads PDFs with `st.file_uploader`.
  - Builds or retrieves the FAISS index using `@st.cache_resource` in `create_vectordb(...)`.
  - On a question:
    - Retrieves top-k chunks with `vectordb.similarity_search(...)`.
    - Injects those chunks into a system prompt.
    - Streams model output using OpenAI’s v1 Chat Completions API.

- `brain.py`
  - `parse_pdf(...)`: uses `pypdf` to extract text from each page and normalizes whitespace/hyphenation.
  - `text_to_docs(...)`: splits text into chunks using `RecursiveCharacterTextSplitter`, attaches `filename`, `page`, `chunk` metadata.
  - `docs_to_index(...)`: creates a FAISS index via `FAISS.from_documents(...)` with `OpenAIEmbeddings`.
  - `get_index_for_pdf(...)`: orchestrates PDF parse → chunk → embed → FAISS index.

### Data flow of a query

1. Upload PDFs → `parse_pdf` extracts text per page.
2. Text → `text_to_docs` creates chunked `Document` objects with metadata.
3. Docs → `docs_to_index` embeds with `OpenAIEmbeddings` and builds a FAISS index.
4. On a user question → `similarity_search(k=3)` returns the most relevant chunks.
5. App forms a system prompt with those chunks and streams a response from the model.
6. The UI displays tokens as they arrive.

### Security and limits

- Your OpenAI key is required; store it in `.streamlit/secrets.toml` or environment.
- Uploaded files stay in memory and are not written to disk unless you add persistence.
- FAISS index is in-memory unless you add the optional save/load shown above.

### Updated README content you can paste

```markdown
<code_block_to_apply_changes_from>
```

### Run
- Set `OPENAI_API_KEY` in `.streamlit/secrets.toml` or as an env var.
- `pip install -r requirements.txt`
- `streamlit run app.py`
```

- Want me to apply these README edits and add the optional persistence helpers to `brain.py` for you?