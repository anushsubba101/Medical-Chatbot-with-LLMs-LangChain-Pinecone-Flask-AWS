# 🩺 Medical Chatbot with LLMs, LangChain, Pinecone, Flask & Render

A Retrieval-Augmented Generation (RAG) medical Q&A chatbot. It answers health-related questions by retrieving relevant passages from a medical reference book and generating concise, grounded answers with an LLM — served through a Flask web app with a simple chat UI, deployed on **Render**.

---

## 📖 About

This project ingests a medical textbook (PDF), splits it into chunks, embeds those chunks, and stores the embeddings in a **Pinecone** vector index. At query time, the user's question is embedded, the most relevant chunks are retrieved from Pinecone, and a **Groq**-hosted LLM generates a short, context-grounded answer using LangChain's retrieval-chain utilities. The whole thing is wrapped in a **Flask** app with a browser-based chat interface.

---

## ✨ Features

- 📄 **PDF ingestion pipeline** — loads a medical reference PDF (`data/Medical_book.pdf`), splits it into overlapping text chunks (`RecursiveCharacterTextSplitter`), and filters document metadata down to just the source.
- 🧠 **Embeddings** — uses HuggingFace's hosted Inference API (`sentence-transformers/all-MiniLM-L6-v2`) rather than a local model, keeping memory usage low for lightweight/free-tier hosting.
- 📦 **Vector storage & retrieval** — chunks are embedded and upserted into a **Pinecone** serverless index (`medical-chatbot`, cosine similarity, 384 dimensions); retrieval pulls the top-k (3) most relevant chunks per query.
- 🤖 **LLM-powered answers** — uses `ChatGroq` (`llama-3.3-70b-versatile`) with a system prompt that instructs the model to answer only from retrieved context, admit when it doesn't know, and keep answers to three sentences or fewer.
- 🔗 **LangChain retrieval chain** — built with `create_retrieval_chain` + `create_stuff_documents_chain` to combine retrieval and generation.
- 💬 **Web chat UI** — a Flask app (`app.py`) serving a Bootstrap-based chat page (`templates/chat.html`, `static/style.css`) with a `/get` endpoint for message exchange.
- ☁️ **AWS-ready** — structured as a standard Python package (`setup.py`, `runtime.txt`) suitable for deployment on AWS (e.g. EC2/Elastic Beanstalk) behind `gunicorn`.

---

## 📂 Project Structure

```
Medical-Chatbot-with-LLMs-LangChain-Pinecone-Flask-Render/
├── app.py                  # Flask app: loads the Pinecone index, builds the RAG chain, serves the chat UI
├── store_index.py           # One-time ingestion script: PDF → chunks → embeddings → Pinecone index
├── src/
│   ├── helper.py           # PDF loading, chunking, and embeddings helper functions
│   └── prompt.py            # System prompt for the medical Q&A assistant
├── templates/
│   └── chat.html            # Chat UI (Bootstrap + jQuery)
├── static/
│   └── style.css             # Chat UI styling
├── data/
│   └── Medical_book.pdf      # Source medical reference document
├── research/
│   └── trials.ipynb          # Exploratory notebook used during development
├── requirements.txt
├── runtime.txt               # Python version pin (3.11.9)
├── setup.py                  # Local package setup
└── LICENSE
```

---

## 🛠️ Tech Stack

- **Python 3.11**
- **[LangChain](https://python.langchain.com/)** (`langchain`, `langchain-classic`, `langchain-core`, `langchain-community`, `langchain-text-splitters`) — chaining, document loading, and retrieval
- **[LangChain Groq](https://python.langchain.com/docs/integrations/chat/groq/)** — LLM inference via Groq
- **[LangChain Pinecone](https://python.langchain.com/docs/integrations/vectorstores/pinecone/)** / **[Pinecone](https://www.pinecone.io/)** — vector database for storing and retrieving document embeddings
- **[LangChain HuggingFace](https://python.langchain.com/docs/integrations/text_embedding/huggingface/)** — hosted embedding inference
- **[Flask](https://flask.palletsprojects.com/)** + **Gunicorn** — web app and production server
- **PyPDF** — PDF parsing
- **[Render](https://render.com/)** — deployment platform

---

## 🚀 Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/anushsubba101/Medical-Chatbot-with-LLMs-LangChain-Pinecone-Flask-Render.git
cd Medical-Chatbot-with-LLMs-LangChain-Pinecone-Flask-Render
```

### 2. Create a virtual environment
```bash
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Set up environment variables
Create a `.env` file in the project root:
```env
PINECONE_API_KEY=your_pinecone_api_key_here
GROQ_API_KEY=your_groq_api_key_here
```
> `store_index.py` also references `OPENAI_API_KEY` — add it to your `.env` if you keep that line, or remove it if you're not using OpenAI.

### 5. Build the Pinecone index
This ingests `data/Medical_book.pdf`, chunks it, embeds it, and creates/populates the Pinecone index. Run it once (or whenever the source document changes):
```bash
python store_index.py
```

### 6. Run the app
```bash
python app.py
```
The app runs on `http://0.0.0.0:8080` by default. Open it in your browser and start chatting.

For production, serve it with Gunicorn:
```bash
gunicorn -b 0.0.0.0:8080 app:app
```

---

## ☁️ Deploying to Render

This app is deployed as a **Render Web Service**. To deploy your own copy:

1. Push the repo to GitHub and create a new **Web Service** on [Render](https://render.com/), pointing it at your repo.
2. Set the **Build Command**:
   ```bash
   pip install -r requirements.txt
   ```
3. Set the **Start Command**:
   ```bash
   gunicorn app:app
   ```
4. Add the following **Environment Variables** in the Render dashboard:
   - `PINECONE_API_KEY`
   - `GROQ_API_KEY`
5. Make sure your Pinecone index has already been created and populated (run `python store_index.py` locally, or as a one-off Render job) before the app goes live — `app.py` expects the `medical-chatbot` index to already exist.

Render assigns a port automatically via the `PORT` environment variable; if you want to respect that rather than the hardcoded `8080` in `app.py`, update the `app.run(...)` call to use `port=int(os.environ.get("PORT", 8080))`.

---

## 📌 Prerequisites

- Python 3.11
- A [Pinecone](https://www.pinecone.io/) account and API key
- A [Groq](https://console.groq.com/) API key

---

## ⚠️ Disclaimer

This chatbot is a demonstration project for educational purposes. It answers based on a single reference PDF and is **not** a substitute for professional medical advice, diagnosis, or treatment.

---

## 📄 License

See [`LICENSE`](./LICENSE) for details.
