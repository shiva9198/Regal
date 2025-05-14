# Regality AI — Graph RAG on FEMA & RBI Compliance

## 📄 Description

**Regality AI** is an advanced AI system purpose-built to offer expert-level guidance on cross-border financial transactions governed by Indian regulations, including the Foreign Exchange Management Act (FEMA) and directions from the Reserve Bank of India (RBI). The bot is tailored to assist professionals, compliance officers, and regulatory teams in decoding complex legal provisions and making informed decisions on:

- **Overseas Direct Investment (ODI)**
- **Foreign Direct Investment (FDI)**
- **Export/Import and Current Account Transactions**
- **FEMA Violations and Compounding Procedures**
- **RBI Master Directions, Notifications, and Circulars**

We combine **Graph-based Knowledge Representation** with **Retrieval-Augmented Generation (RAG)** to deliver grounded, traceable, and regulation-specific answers—rooted directly in official RBI and FEMA texts.

---

## 📌 Project Overview

We are leveraging the [Milvus vector database](https://milvus.io/) to store and retrieve document embeddings, combined with a knowledge graph that encodes domain relationships under FEMA/RBI regulations. These components together form the basis of a **Graph RAG** architecture that enhances factual accuracy and regulatory reasoning.

> **Live documentation reference:**  
> [Graph RAG with Milvus](https://milvus.io/docs/graph_rag_with_milvus.md)

---

## 🏗️ System Architecture

User Query
│
▼
[Query Encoder (LLM)]
│
▼
[Retriever]
├──> [Vector DB (Milvus)]
└──> [Knowledge Graph]
│
▼
[Response Generator (LLM with RAG)]
│
▼
Final Answer with Source Traceability


---

## 🧠 Core Features

- ✅ **Regulatory-focused RAG**: Incorporates entities and relationships from FEMA and RBI documents
- ✅ **Graph-Aided Retrieval**: Queries enriched by graph traversal and semantic proximity
- ✅ **Legal Traceability**: Citations and source references in responses
- ✅ **Modular Setup**: Easy plug-and-play of embeddings, graph logic, and LLM endpoints

---

## 🔧 Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/regality-ai.git
cd regality-ai



---

## 🧠 Core Features

- ✅ **Regulatory-focused RAG**: Incorporates entities and relationships from FEMA and RBI documents
- ✅ **Graph-Aided Retrieval**: Queries enriched by graph traversal and semantic proximity
- ✅ **Legal Traceability**: Citations and source references in responses
- ✅ **Modular Setup**: Easy plug-and-play of embeddings, graph logic, and LLM endpoints

---

## 🔧 Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/regality-ai.git
cd regality-ai


2. Create a Virtual Environment
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt


3. Start Milvus via Docker
docker-compose up -d

4. Load FEMA/RBI Documents
python scripts/extract_triples.py
python scripts/embed_documents.py

Key Directories
| Path         | Description                                        |
| ------------ | -------------------------------------------------- |
| `data/`      | Regulatory document corpus (PDFs, scraped content) |
| `scripts/`   | Preprocessing utilities for KG and embeddings      |
| `graph/`     | GraphDB models and Neo4j integration (if used)     |
| `rag/`       | RAG orchestration components                       |
| `app/`       | FastAPI or Flask backend exposing inference routes |
| `notebooks/` | Experiments, test queries, pipeline validation     |