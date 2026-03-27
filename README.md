# Configurable Multi-Tenant Conversational RAG Platform

A production-style **multi-tenant Retrieval-Augmented Generation (RAG)** platform where **organizations (admins)** can create **role-specific topics/agents**, upload proprietary documents, and give **employees/users** access to chat with **citation-grounded** answers.

The system supports:
- **Per-topic document isolation** using **Pinecone namespaces**
- **Per-user chat history** stored in **PostgreSQL**
- **Hybrid conversational behavior**:
  - Small talk 
  - Memory questions from chat history (“what was my first question?”)
  - RAG answers grounded in uploaded docs  (with citations)
  - Safe fallback when documents don’t contain the answer 

---

## Key Features

###  Multi-Tenant Topics (Role-Specific Agents)
- Admin creates **topics** like “HR Policy”, “Engineering Wiki”, “Onboarding”, etc.
- Each topic has a **behavior prompt** that defines role, tone, and rules.
- **Employees only see topics they are added to**.

###  Secure Admin + User Modes (Product-Style)
- **Admin mode**:
  - One-time bootstrap “Create First Admin”
  - Admin login via token (`X-Admin-Token`)
  - Create topics, upload documents, add employees to topics
- **User mode**:
  - Login via username
  - Select topic
  - Chat with RAG + memory + fallback

###  Citation-Grounded Answers (RAG)
- Documents are chunked and embedded with **MiniLM**.
- Retrieval via **Pinecone vector search**.
- Results reranked with **Cross-Encoder (ms-marco-MiniLM-L-6-v2)**.
- Final answer includes **citations** (source + page when available).

###  Conversation Memory (PostgreSQL)
- Stores per-user (and per-topic) chat history.
- Supports memory-only questions like:
  - “What was my first question?”
  - “Summarize our conversation”
  - “What did I say last time?”

### Safe Fallback (When Docs Don’t Contain Answer)
When context doesn’t contain the answer:
> “I don’t know this based on the provided documents. But here is what I know about it: …”

This makes the chatbot useful even when retrieval is empty, while remaining honest.

---

## Tech Stack

**Backend**
- FastAPI
- PostgreSQL (chat history, users, topics, membership, admin sessions)
- Pinecone (vector DB + namespaces per topic)
- LangChain + Gemini (response generation)
- HuggingFace MiniLM embeddings
- Cross-Encoder reranker

**Frontend**
- Streamlit (Admin/User UI)

**Evaluation**
- `eval.py` to compute:
  - Recall@5 / Recall@10
  - MRR
  - Retrieval latency & response latency
  - (Optional) citation hit + semantic similarity when Gemini quota allows

---

## Architecture Overview

**Ingestion**
1. Admin uploads PDFs/TXTs to a topic
2. Backend extracts text → chunks (RecursiveCharacterTextSplitter)
3. Embeddings created via MiniLM
4. Upsert vectors into Pinecone under the topic’s namespace

**Query**
1. User selects topic and asks a question
2. Intent routing:
   - Small talk → direct response
   - Memory → answers only from chat history
   - Knowledge → retrieval + rerank + RAG answer
3. RAG response returns JSON with answer + citations
4. Postgres stores chat history

---

## Project Structure
# Configurable Multi-Tenant Conversational RAG Platform

A production-style **multi-tenant Retrieval-Augmented Generation (RAG)** platform where **organizations (admins)** can create **role-specific topics/agents**, upload proprietary documents, and give **employees/users** access to chat with **citation-grounded** answers.

The system supports:
- **Per-topic document isolation** using **Pinecone namespaces**
- **Per-user chat history** stored in **PostgreSQL**
- **Hybrid conversational behavior**:
  - Small talk 
  - Memory questions from chat history  (“what was my first question?”)
  - RAG answers grounded in uploaded docs  (with citations)
  - Safe fallback when documents don’t contain the answer 

---

## Key Features

###  Multi-Tenant Topics (Role-Specific Agents)
- Admin creates **topics** like “HR Policy”, “Engineering Wiki”, “Onboarding”, etc.
- Each topic has a **behavior prompt** that defines role, tone, and rules.
- **Employees only see topics they are added to**.

###  Secure Admin + User Modes (Product-Style)
- **Admin mode**:
  - One-time bootstrap “Create First Admin”
  - Admin login via token (`X-Admin-Token`)
  - Create topics, upload documents, add employees to topics
- **User mode**:
  - Login via username
  - Select topic
  - Chat with RAG + memory + fallback

###  Citation-Grounded Answers (RAG)
- Documents are chunked and embedded with **MiniLM**.
- Retrieval via **Pinecone vector search**.
- Results reranked with **Cross-Encoder (ms-marco-MiniLM-L-6-v2)**.
- Final answer includes **citations** (source + page when available).

###  Conversation Memory (PostgreSQL)
- Stores per-user (and per-topic) chat history.
- Supports memory-only questions like:
  - “What was my first question?”
  - “Summarize our conversation”
  - “What did I say last time?”

###  Safe Fallback (When Docs Don’t Contain Answer)
When context doesn’t contain the answer:
> “I don’t know this based on the provided documents. But here is what I know about it: …”

This makes the chatbot useful even when retrieval is empty, while remaining honest.

---

## Tech Stack

**Backend**
- FastAPI
- PostgreSQL (chat history, users, topics, membership, admin sessions)
- Pinecone (vector DB + namespaces per topic)
- LangChain + Gemini (response generation)
- HuggingFace MiniLM embeddings
- Cross-Encoder reranker

**Frontend**
- Streamlit (Admin/User UI)

**Evaluation**
- `eval.py` to compute:
  - Recall@5 / Recall@10
  - MRR
  - Retrieval latency & response latency
  - (Optional) citation hit + semantic similarity when Gemini quota allows

---

## Architecture Overview

**Ingestion**
1. Admin uploads PDFs/TXTs to a topic
2. Backend extracts text → chunks (RecursiveCharacterTextSplitter)
3. Embeddings created via MiniLM
4. Upsert vectors into Pinecone under the topic’s namespace

**Query**
1. User selects topic and asks a question
2. Intent routing:
   - Small talk → direct response
   - Memory → answers only from chat history
   - Knowledge → retrieval + rerank + RAG answer
3. RAG response returns JSON with answer + citations
4. Postgres stores chat history

---

## Project Structure
RAGBOT/
backend/
main.py # FastAPI backend (auth, topics, query pipeline)
admin_auth.py # admin hashing/token/session utilities
create_tables.py # DB schema creation
db.py # DB helpers (if used)
guardrails/ # grounded policy checks
retrieval/ # retrieval utilities (optional)
requirements.txt
frontend/
app.py # Streamlit UI (Admin/User)
eval.py # evaluation script
eval_set*.json # evaluation datasets


---

## Setup Instructions

### 1) Create Python environment
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r backend/requirements.txt

## **2)Environment Variables**

Create a `.env` file in the project root and add:

```env
# Database
DATABASE_URL=your_postgresql_connection_string

# Pinecone
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX_NAME=your_index_name
PINECONE_CLOUD=aws
PINECONE_REGION=us-east-1

# BM25 index path
BM25_PATH=bm25.json

# Gemini
GOOGLE_API_KEY=your_google_api_key

##**3)Create DB tables**
python backend/create_tables.py

##**4)Run backend**
python -m uvicorn backend.main:app --reload --port 8000

##**5)Run frontend**
streamlit run frontend/app.py

##**How to Use (Demo Flow)**
1.Admin Flow

2.Choose Admin

3.If first time → Create First Admin

4.Create a topic + behavior prompt 

5.You have a option to improe the prompt , so the prompt can be improved

6.Upload PDF/TXT documents into that topic

7.Add employees to the topic by username

##**User Flow**
1.Choose Employee/User

2.Login with username

3.Select topic

4.Ask questions:

  *Small talk: “hi”

  *RAG: “What is the perinuclear space?”

  *Memory: “What was my first question?”

##**Evaluation**
**Option A (Retrieval-Only Evaluation) ✅ Most reliable**
source venv/bin/activate
export USER_ID="User ID"
export TOPIC_ID="Topic ID"
export EVAL_SET=eval_set_nucleus.json
export SKIP_QUERY=1
python eval.py

**Option B (Full Pipeline Evaluation: retrieval + answer)**
export SKIP_QUERY=0
export TIMEOUT=180
export ANSWER_DELAY_SEC=5
python eval.py

## Demo (Screenshots)

This section walks through the end-to-end flow: admin setup → topic creation → document ingestion → user chat → Pinecone verification → evaluation results.

> All screenshots are stored in the `screenshots/` folder  


### 1) App Landing / Role Selection
Shows the starting page where users choose between **Admin** and **Employee/User** mode.
![Landing](screenshots/01_landing.png)

### 2) Admin Login
Admin authentication screen used to access protected actions like creating topics, uploading documents, and adding users.
![Admin Login](screenshots/02_admin%20login%20page.png)

### 3) Admin Dashboard (Overview)
Admin dashboard entry point showing the main workflow sections:
- Create Topic
- Select Topic
- Upload Documents
- Add Employee/User to Topic
![Admin Dashboard](screenshots/03_admin%20dashboard%201.png)

### 4) Create Topic (Draft Prompt)
Creating a new topic (tenant workspace) with a **topic behavior prompt** that controls how the agent responds.
![Create Topic - Draft Prompt](screenshots/04_admin%20dashboard%202%20(creating%20topic).png)

### 5) Create Topic (Topic Selected)
Topic is configured and selected, ready for ingestion and membership assignment.
![Create Topic - Selected](screenshots/05_admin%20dashboard%203(creating%20topic).png)

### 6) Improve Prompt (Prompt Engineering)
The system improves the admin’s draft into a stronger **topic behavior prompt** (grounded, citation-required, safe).
![Improve Prompt](screenshots/06_admin%20dashboard%204%20(improving%20prompt).png)

### 7) Upload Documents (Ingestion → Pinecone)
Admin uploads PDFs/TXT files into the selected topic. The backend chunks, embeds, and upserts vectors into the topic namespace.
![Upload Documents](screenshots/07_admin%20dashboard%20(Uploading%20Documents).png)

### 8) User Login
Employee/User logs in and can access only topics they are a member of.
![User Login](screenshots/08_User%20login%20Page.png)

### 9) User Chat (RAG + Citations + Fallback)
Demonstrates:
- Small talk handling
- Topic-grounded Q&A using uploaded documents
- Safe fallback when the answer is not found in the documents
![User Chat](screenshots/09_User%20Chat%20with%20Ragbot.png)

### 10) Pinecone Dashboard (Index View)
Shows the Pinecone index used to store dense vectors for retrieval.
![Pinecone Dashboard](screenshots/10_Pinecone%20dashboard%201.png)

### 11) Pinecone Namespace (Per-Topic Isolation)
Each topic uses a separate **Pinecone namespace** (e.g., `topic-ncleus-...`) to isolate tenant data and prevent cross-topic leakage.
![Pinecone Namespace](screenshots/11_Pinecone%20namespace.png)

---

## Evaluation

Evaluation is run using `eval.py` against a curated question set (example: nucleus topic).  
Metrics include:
- **Recall@5 / Recall@10** (retrieval success)
- **MRR** (ranking quality)
- Optional: citation hit rate + semantic similarity (when answer generation is enabled)

### 12) Evaluation Results (Retrieval-Only) — Run 1
Retrieval-only mode (no LLM calls) to avoid LLM quota limits and isolate retrieval quality.
![Eval Retrieval Only - 1](screenshots/12_Evaluation%20Results%201(Retrieval%20only).png)

### 13) Evaluation Results (Retrieval-Only) — Run 2
Additional retrieval run confirming stable retrieval metrics across the dataset.
![Eval Retrieval Only - 2](screenshots/13_Evaluation%20Results%202(Retrieval%20only).png)

### 14) Evaluation Results (Retrieval + Semantic Similarity)
Full mode example showing retrieval + generated answers compared against expected answers using semantic similarity.
![Eval Retrieval + Similarity](screenshots/14_Evaluation%20Results%20(retrieval%20+Semantic%20Similarity).png)
