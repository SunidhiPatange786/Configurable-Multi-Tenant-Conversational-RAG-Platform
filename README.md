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
