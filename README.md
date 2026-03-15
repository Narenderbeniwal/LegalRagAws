<div align="center">

# ⚖️ Legal RAG AWS

### AI-Powered Legal Document Intelligence on AWS

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.9+-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" />
  <img src="https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white" />
  <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=FF9900" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/AWS_Bedrock-Titan_Embed_%26_LLM-FF9900?style=flat-square&logo=amazon-aws" />
  <img src="https://img.shields.io/badge/OpenSearch-Vector_Store-005EB8?style=flat-square&logo=opensearch" />
  <img src="https://img.shields.io/badge/S3-Document_Storage-569A31?style=flat-square&logo=amazon-s3" />
</p>

<br/>

> **Upload legal documents. Ask questions. Get cited answers — instantly.**
>
> A production-ready Retrieval-Augmented Generation (RAG) system that combines
> semantic vector search, keyword search, and AWS Bedrock LLMs to deliver
> precise, evidence-backed answers from your legal document library.

<br/>

</div>

---

## 📋 Table of Contents

- [✨ Features](#-features)
- [🏗️ Architecture](#️-architecture)
- [🔄 How It Works](#-how-it-works)
- [🛠️ Tech Stack](#️-tech-stack)
- [📁 Project Structure](#-project-structure)
- [⚡ Quick Start](#-quick-start)
- [🔧 Configuration](#-configuration)
- [🚀 Running the Application](#-running-the-application)
- [📡 API Reference](#-api-reference)
- [🧪 Testing](#-testing)
- [💰 Cost Estimation](#-cost-estimation)
- [🔐 IAM Permissions](#-iam-permissions)

---

## ✨ Features

<table>
<tr>
<td width="50%">

**📄 Document Processing**
- Upload PDF and DOCX legal documents
- Recursive chunking with configurable overlap
- Section header extraction & metadata tagging
- SHA-256 file-level deduplication

</td>
<td width="50%">

**🔍 Intelligent Search**
- Semantic k-NN vector search (1536-dim embeddings)
- BM25 keyword full-text search
- Hybrid reranking with tunable alpha
- Chunk-level deduplication

</td>
</tr>
<tr>
<td width="50%">

**🤖 AI-Powered Answers**
- Context-aware answers via AWS Bedrock LLM
- Every answer includes source citations
- File name, page number, relevance score & snippet
- 32K context window support

</td>
<td width="50%">

**☁️ AWS-Native Infrastructure**
- Fully managed services — zero self-hosting
- S3 with versioning & AES-256 encryption
- OpenSearch with k-NN plugin
- Bedrock Titan Embed + Text Premier

</td>
</tr>
</table>

---

## 🏗️ Architecture

```
╔══════════════════════════════════════════════════════════════╗
║                  🖥️  Streamlit Frontend                      ║
║              (Upload UI · Chat Interface)                    ║
╚══════════════════════╤═══════════════════════════════════════╝
                       │  REST API
╔══════════════════════▼═══════════════════════════════════════╗
║                   ⚡ FastAPI Backend                          ║
║                                                              ║
║   ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  ║
║   │ POST         │  │ POST         │  │ GET              │  ║
║   │ /api/upload  │  │ /api/chat    │  │ /health          │  ║
║   └──────┬───────┘  └──────┬───────┘  └──────────────────┘  ║
║          │                 │                                  ║
║   ┌──────▼─────────────────▼──────────────────────────────┐  ║
║   │                   Services Layer                       │  ║
║   │  📄 Document Processor  →  🔢 Embedding Service        │  ║
║   │  🔍 OpenSearch Service  →  ♻️  Dedup Middleware         │  ║
║   │  🎯 Reranker            →  💬 LLM Service              │  ║
║   └────────────────────────────────────────────────────────┘  ║
╚══════════╤════════════════╤════════════════╤══════════════════╝
           │                │                │
    ╔══════▼══════╗  ╔══════▼══════╗  ╔══════▼══════╗
    ║   🪣 AWS S3  ║  ║ 🔎 OpenSearch║  ║ 🤖 Bedrock  ║
    ║  Document   ║  ║  Vector &   ║  ║  Titan Embed║
    ║  Storage    ║  ║  Keyword    ║  ║  + LLM      ║
    ╚═════════════╝  ╚═════════════╝  ╚═════════════╝
```

---

## 🔄 How It Works

### 📥 Pipeline 1 — Document Ingestion

```
 ┌──────────┐    ┌─────────────┐    ┌──────────┐    ┌──────────────┐
 │  Upload   │───▶│ SHA-256     │───▶│  Store   │───▶│ Parse Text   │
 │ PDF/DOCX  │    │ Dedup Check │    │  in S3   │    │ PDF / DOCX   │
 └──────────┘    └─────────────┘    └──────────┘    └──────┬───────┘
                                                           │
 ┌──────────────────────────────────────────────────────────▼───────┐
 │  Recursive Chunk  →  Extract Headers  →  Tag Metadata            │
 │  (1000 chars, 200 overlap)                                        │
 └──────────────────────────────────┬───────────────────────────────┘
                                    │
 ┌──────────────────────────────────▼───────────────────────────────┐
 │  Titan Embed (1536-dim)  →  Bulk Index in OpenSearch             │
 │  + Chunk-level dedup before each insert                          │
 └──────────────────────────────────────────────────────────────────┘
```

### 💬 Pipeline 2 — Query & Answer

```
 ┌──────────┐    ┌─────────────┐         ┌──────────────────────────┐
 │  User    │───▶│ Titan Embed │────┬───▶│  Semantic k-NN (top 10)  │
 │  Query   │    │   Query     │    │    └──────────────────────────┘
 └──────────┘    └─────────────┘    │    ┌──────────────────────────┐
                                    └───▶│  Keyword BM25  (top 10)  │
                                         └────────────┬─────────────┘
                                                      │
                              ┌───────────────────────▼──────────────┐
                              │  Hybrid Rerank                        │
                              │  score = 0.7 × semantic               │
                              │        + 0.3 × keyword                │
                              │  → Top 5 deduplicated chunks          │
                              └───────────────────────┬──────────────┘
                                                      │
                              ┌───────────────────────▼──────────────┐
                              │  Bedrock LLM  →  Cited Answer        │
                              │  (source · page · score · snippet)   │
                              └──────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|:------|:-----------|:--------|
| 🖥️ **Frontend** | Streamlit | Document upload & chat UI |
| ⚡ **Backend** | FastAPI + Uvicorn | REST API server |
| 🤖 **Embeddings** | AWS Bedrock — `amazon.titan-embed-text-v1` | 1536-dim vector generation |
| 💬 **LLM** | AWS Bedrock — `amazon.titan-text-premier-v1:0` | Answer synthesis (32K ctx) |
| 🔎 **Vector Store** | AWS OpenSearch (k-NN plugin) | Semantic + keyword search |
| 🪣 **Object Storage** | AWS S3 | Encrypted document storage |
| 📄 **Doc Parsing** | PyPDF2, python-docx | PDF & DOCX text extraction |
| 🔢 **Tokenization** | tiktoken | Token counting & chunking |
| 🔐 **Auth** | requests-aws4auth | AWS Signature v4 signing |
| ✅ **Validation** | Pydantic v2 | Request/response schemas |

---

## 📁 Project Structure

```
LegalRagAws/
│
├── 🐍 backend/
│   ├── main.py                    ← FastAPI app entry point
│   ├── config.py                  ← Centralized config (env vars)
│   │
│   ├── models/
│   │   ├── document.py            ← Document & chunk schemas
│   │   └── chat.py                ← Chat request/response models
│   │
│   ├── routes/
│   │   ├── upload.py              ← POST /api/upload
│   │   └── chat.py                ← POST /api/chat
│   │
│   └── services/
│       ├── s3_service.py          ← S3 upload/download/list
│       ├── document_processor.py  ← PDF/DOCX parsing & chunking
│       ├── embedding_service.py   ← Bedrock embedding calls
│       ├── opensearch_service.py  ← Vector store operations
│       ├── dedup_middleware.py    ← Duplicate detection
│       ├── reranker.py            ← Hybrid search reranking
│       └── llm_service.py        ← Bedrock LLM answer generation
│
├── 🎨 frontend/
│   └── app.py                     ← Streamlit web UI
│
├── ☁️ infrastructure/
│   ├── setup_all.py               ← Orchestrator (run this first)
│   ├── setup_s3.py                ← S3 bucket provisioning
│   ├── setup_opensearch.py        ← OpenSearch domain creation
│   └── setup_bedrock.py           ← Bedrock model verification
│
├── 🧪 tests/
│   ├── test_ingestion.py          ← Document pipeline tests
│   ├── test_query.py              ← Query pipeline tests
│   └── test_dedup.py              ← Deduplication logic tests
│
├── 📂 sample_documents/           ← Sample legal docs for testing
├── 📄 requirements.txt
├── 💰 COST.md                     ← AWS cost breakdown (INR)
└── 🔒 .env                        ← Environment variables (git-ignored)
```

---

## ⚡ Quick Start

### Prerequisites

- Python **3.9+**
- AWS account with access to **Bedrock**, **S3**, and **OpenSearch**
- Bedrock model access enabled for:
  - `amazon.titan-embed-text-v1`
  - `amazon.titan-text-premier-v1:0`

---

### Step 1 — Clone

```bash
git clone <repository-url>
cd LegalRagAws
```

### Step 2 — Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 3 — Configure Environment

Create a `.env` file in the project root:

```env
# ─── AWS Credentials ────────────────────────────────────────────
AWS_ACCESS_KEY_ID=your_access_key_here
AWS_SECRET_ACCESS_KEY=your_secret_key_here
AWS_REGION=us-east-1

# ─── S3 Storage ─────────────────────────────────────────────────
S3_BUCKET_NAME=legal-rag-documents

# ─── OpenSearch ─────────────────────────────────────────────────
OPENSEARCH_DOMAIN_NAME=legal-rag-search
OPENSEARCH_ENDPOINT=               # Auto-populated after setup
OPENSEARCH_MASTER_USER=admin
OPENSEARCH_MASTER_PASSWORD=YourP@ssw0rd   # 8+ chars, mixed, number, special

# ─── Bedrock Models ─────────────────────────────────────────────
BEDROCK_EMBEDDING_MODEL_ID=amazon.titan-embed-text-v1
BEDROCK_LLM_MODEL_ID=amazon.titan-text-premier-v1:0
```

### Step 4 — Provision AWS Infrastructure

```bash
python infrastructure/setup_all.py
```

> ⏱️ **Note:** OpenSearch domain creation takes ~15–20 minutes on first run.

This script will:
- ✅ Create S3 bucket with versioning & AES-256 encryption
- ✅ Provision OpenSearch domain with k-NN plugin
- ✅ Verify Bedrock model access
- ✅ Auto-populate `OPENSEARCH_ENDPOINT` in your `.env`

---

## 🚀 Running the Application

**Terminal 1 — Start Backend**
```bash
uvicorn backend.main:app --reload --port 8000
```
Backend → `http://localhost:8000` · Swagger Docs → `http://localhost:8000/docs`

**Terminal 2 — Start Frontend**
```bash
streamlit run frontend/app.py
```
Frontend → `http://localhost:8501`

---

## 📡 API Reference

### `POST /api/upload` — Upload a Document

```bash
curl -X POST http://localhost:8000/api/upload \
  -F "file=@contract.pdf"
```

**Response:**
```json
{
  "message": "Document processed successfully",
  "filename": "contract.pdf",
  "s3_key": "documents/contract.pdf",
  "is_duplicate": false,
  "chunks_created": 42
}
```

---

### `POST /api/chat` — Ask a Question

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"query": "What are the termination clauses?", "top_k": 5, "alpha": 0.7}'
```

**Request Body:**

| Field | Type | Default | Description |
|:------|:-----|:--------|:------------|
| `query` | `string` | required | Natural language question |
| `top_k` | `integer` | `5` | Number of results to return |
| `alpha` | `float` | `0.7` | Semantic weight (`0`=keyword only → `1`=semantic only) |

**Response:**
```json
{
  "answer": "The termination clause states that either party may terminate...",
  "citations": [
    {
      "source": "contract.pdf",
      "page": 3,
      "score": 0.92,
      "snippet": "Either party may terminate this agreement upon 30 days written notice..."
    }
  ],
  "query": "What are the termination clauses?"
}
```

---

### `GET /health` — Health Check

```bash
curl http://localhost:8000/health
```

---

## 🔧 Configuration

| Variable | Default | Description |
|:---------|:--------|:------------|
| `CHUNK_SIZE` | `1000` | Characters per document chunk |
| `CHUNK_OVERLAP` | `200` | Overlap between adjacent chunks |
| `TOP_K` | `5` | Results returned per query |
| `RERANK_ALPHA` | `0.7` | Semantic search weight in hybrid scoring |
| `EMBEDDING_DIMENSION` | `1536` | Titan v1 embedding dimensions |

---

## 🧪 Testing

```bash
# Run all tests
python -m pytest tests/

# Individual suites
python -m pytest tests/test_ingestion.py   # Document processing pipeline
python -m pytest tests/test_query.py       # Query & retrieval pipeline
python -m pytest tests/test_dedup.py       # Deduplication logic
```

> 📂 Sample legal documents are available in `sample_documents/` for end-to-end testing.

---

## 💰 Cost Estimation

Estimated monthly AWS costs for **1,000 users × 50 queries/day**:

| Service | Dev Setup | Production |
|:--------|----------:|----------:|
| 🤖 Bedrock Embeddings | ₹672 | ₹672 |
| 💬 Bedrock LLM (Nova Lite) | ₹21,546 | ₹21,546 |
| 🔎 OpenSearch | ₹2,342 | ₹8,684 |
| 🪣 S3 Storage | ₹30 | ₹30 |
| 🖥️ EC2 Compute | ₹5,102 | ₹14,441 |
| | | |
| **Total** | **~₹29,692/mo** | **~₹44,673/mo** |
| **Per Query** | **~₹0.03** | **~₹0.03** |

> 💡 Potential savings with Nova Micro model, caching & reserved instances: **~₹29,600/mo**

See [COST.md](COST.md) for full breakdown and optimization strategies.

---

## 🔐 IAM Permissions

Minimum required permissions for your AWS user/role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3Access",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::legal-rag-documents/*"
    },
    {
      "Sid": "OpenSearchAccess",
      "Effect": "Allow",
      "Action": "es:*",
      "Resource": "arn:aws:es:*:*:domain/legal-rag-search/*"
    },
    {
      "Sid": "BedrockAccess",
      "Effect": "Allow",
      "Action": "bedrock:InvokeModel",
      "Resource": "*"
    }
  ]
}
```

---

<div align="center">

**Built with ❤️ using AWS Bedrock · OpenSearch · FastAPI · Streamlit**

<br/>

[![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazon-aws&logoColor=FF9900)](https://aws.amazon.com)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white)](https://streamlit.io)

*MIT License · © 2026*

</div>
