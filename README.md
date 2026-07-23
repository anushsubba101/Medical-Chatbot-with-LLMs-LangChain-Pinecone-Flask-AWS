# 🤖 LangGraph Chatbot

A conversational AI chatbot built with **LangGraph** and **Groq LLMs**, featuring tool use, multi-turn memory, multi-threaded conversations, and custom **MCP (Model Context Protocol)** tool servers, all wrapped in a Streamlit UI.

---

## 📖 About

This project demonstrates how to build a stateful chatbot using LangGraph's `StateGraph`, going from a minimal single-node chatbot up to a full-featured assistant that can search the web, look up stock prices, track expenses, and do arithmetic, all through pluggable tools.

The repo includes several progressively richer versions of the frontend/backend so you can see how the architecture evolves:

- A basic request/response chatbot
- A streaming version (token-by-token output)
- A multi-thread version (multiple saved conversations)
- A full version combining threading, streaming, tool calls, and persistent (SQLite-backed) memory

---

## ✨ Features

- **LangGraph-powered conversation graph**, a `StateGraph` with a single `chat_node` that invokes the LLM and appends to message history via `add_messages`.
- **Groq LLM** — powered by `llama-3.3-70b-versatile` via `langchain_groq`.
- **Conversation memory** — `InMemorySaver` for simple in-memory checkpointing, and `AsyncSqliteSaver` for persistent, multi-thread conversation history stored in SQLite.
- **Multiple conversation threads** — start new chats and switch back to previous ones (thread IDs generated with `uuid`).
- **Streaming responses** — assistant replies stream token-by-token into the Streamlit UI.
- **Tool calling** — the LLM can invoke:
  - 🔎 **Web search** via `DuckDuckGoSearchRun`
  - 📈 **Stock price lookup** via the Alpha Vantage API
  - 🧮 **Arithmetic operations** (add, subtract, multiply, divide, power, modulus) via a custom **MCP server** (`main.py`, built with `fastmcp`)
  - 💰 **Expense tracking** (add / list / summarize expenses in a SQLite database) via a second custom **MCP server** (`mcp_server.py`), connected as a remote MCP tool via `MultiServerMCPClient`
- **Streamlit frontends** - several UI variants (`frontend.py`, `frontend-streamlit.py`, `frontend_streaming.py`, `frontend_threading.py`) showing the progression from simple to full-featured.

---

## 📂 Project Structure

```
Langgraph-Chatbot/
├── backend.py               # Minimal LangGraph chatbot (in-memory checkpointing)
├── database.py               # Full backend: SQLite memory, tools, MCP client, stock/search tools
├── main.py                   # MCP server exposing arithmetic tools (fastmcp)
├── mcp_server.py              # MCP server exposing an expense-tracker (add/list/summarize)
├── frontend-streamlit.py      # Simplest Streamlit chat UI (uses backend.py)
├── frontend_streaming.py      # Adds token-by-token streaming (uses backend.py)
├── frontend_threading.py      # Adds multiple conversation threads (uses backend.py)
├── frontend.py                 # Full UI: threading + streaming + persistent memory (uses database.py)
└── requirements.txt
```

---

## 🛠️ Tech Stack

- **Python**
- **[LangGraph](https://github.com/langchain-ai/langgraph)** — state graph orchestration & checkpointing (`InMemorySaver`, `AsyncSqliteSaver`)
- **[LangChain](https://github.com/langchain-ai/langchain)** — messages, tools, and community integrations
- **[Groq](https://groq.com/)** (`langchain_groq`) — fast LLM inference (`llama-3.3-70b-versatile`)
- **[FastMCP](https://github.com/jlowin/fastmcp)** — for building the custom MCP tool servers
- **[Streamlit](https://streamlit.io/)** — chat UI
- **SQLite** — persistent conversation memory and expense storage
- **DuckDuckGo Search** & **Alpha Vantage API** — external data tools

---

## 🚀 Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/anushsubba101/Langgraph-Chatbot.git
cd Langgraph-Chatbot
```

### 2. Create a virtual environment
```bash
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
pip install langgraph langchain langchain-groq langchain-community langchain-mcp-adapters streamlit python-dotenv aiosqlite fastmcp requests
```
> `requirements.txt` currently only lists `fastmcp` — the command above adds the rest of what the code imports. Consider updating `requirements.txt` to pin all of these for reproducibility.

### 4. Set up environment variables
Create a `.env` file in the project root:
```env
GROQ_API_KEY=your_groq_api_key_here
```

### 5. (Optional) Run the MCP tool servers
The chatbot expects an expense-tracker MCP server reachable over HTTP (by default pointed at a deployed Railway URL in `database.py`). To run your own locally:
```bash
python mcp_server.py      # expense-tracker MCP server
python main.py             # arithmetic MCP server
```
Update the MCP client URL in `database.py` if you're hosting the server yourself.

### 6. Run the chatbot UI
Pick whichever frontend you want to try:
```bash
streamlit run frontend-streamlit.py   # simplest version
streamlit run frontend_streaming.py   # with streaming
streamlit run frontend_threading.py   # with multi-thread chats
streamlit run frontend.py             # full version (threading + streaming + tools + persistent memory)
```

---

## 📌 Prerequisites

- Python 3.9+
- A [Groq API key](https://console.groq.com/)
- (Optional) An Alpha Vantage API key if you want to use your own key for stock lookups — the code currently includes a hardcoded demo key in `database.py`, which you should replace with your own via an environment variable.

---

## ⚠️ Notes

- `database.py` contains a hardcoded Alpha Vantage API key — move this to an environment variable before deploying or sharing this project publicly.
- The MCP expense-tracker client in `database.py` points to a specific deployed Railway URL by default — swap this for your own MCP server endpoint if you're self-hosting.

---

## 🤝 Contributing

This is primarily a personal learning/portfolio project, but suggestions, issue reports, and pull requests are welcome.

---

## 📄 License

No license has been specified yet. Consider adding one (e.g., MIT) if you'd like others to freely use or build on this code.

---

## ⭐ Acknowledgements

- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [LangChain Documentation](https://python.langchain.com/)
- [Groq](https://groq.com/)
- [FastMCP](https://github.com/jlowin/fastmcp)
