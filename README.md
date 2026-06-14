# Multi-Container DevSecOps & AI Vulnerability Orchestrator

A production-grade, multi-container security platform that automates vulnerability analysis (CVEs), risk scoring, and remediation planning using a multi-agent orchestrated pipeline. 

This repository coordinates the entire local microservice architecture using **Docker Compose**, orchestrating five interconnected services: a React frontend dashboard, a Node.js Express.js API gateway, a MongoDB database, a Qdrant vector database, and a Python FastAPI service for local vector embeddings.

---

## 🛠️ Multi-Container Architecture & Services

The system is configured to launch as a unified stack via `docker-compose.yml`, using Docker's internal DNS resolution and custom virtual network bridges:

```
                            ┌──────────────────────────────────┐
                            │    frontend (React, Port 3000)   │
                            └────────────────┬─────────────────┘
                                             │ (HTTP JSON API)
                                             ▼
                            ┌──────────────────────────────────┐
                            │     backend (Express, Port 5000) │
                            └────┬──────────────┬───────────┬──┘
                                 │              │           │
           (Persist & Retrieve)  │              │           │ (Structured Generation)
                                 ▼              │           ▼
                       ┌───────────┐            │     ┌───────────┐
                       │  mongodb  │            │     │ Groq LLM  │
                       │ (Port     │            │     │   (API)   │
                       │   27017)  │            │     └───────────┘
                       └───────────┘            ▼
                                        ┌───────────────┐
                                        │    qdrant     │ (Vector Indexes)
                                        └───────┬───────┘ (Ports 6333/6334)
                                                │
                                                ▼
                                    ┌──────────────────────┐
                                    │  embedding-service   │
                                    │ (Python, Port 8000)  │
                                    └──────────────────────┘
```

The system components are decoupled into the following Docker containers:

1. **`frontend` (React + Tailwind CSS):** Serves the security operations dashboard on port `3000`. Visualizes high/low severity distributions using Recharts and provides interactive search, filtering, and RAG chat.
2. **`backend` (Node.js/Express.js):** Runs the API gateway and orchestrates AI workflows on port `5000`. Connects to MongoDB, Qdrant, and the local Python embedding model.
3. **`embedding-service` (FastAPI + Sentence-Transformers):** A Python microservice that loads the `BAAI/bge-small-en-v1.5` transformer model, exposing a high-performance vector creation endpoint on port `8000`.
4. **`mongodb` (NoSQL Database):** Persists scanner records and vulnerability logs. Bound to port `27017` with data persisted using a named volume (`mongo-data`).
5. **`qdrant` (Vector Database):** Stores dense document embeddings. Exposes ports `6333` and `6334` for vector search, with data persisted via a named volume (`qdrant-data`).

---

## 🚀 Key Pipelines

### 🤖 Multi-Step CVE Orchestrator
When a new vulnerability scan is received, the backend executes a sequential multi-agent workflow:
* **Analysis:** Extracts structured details from the vulnerability signature based on predefined schemas.
* **Risk Assessment:** Performs business impact analyses and calculates custom risk priority scores.
* **Remediation Planning:** Dynamically generates step-by-step remediation plans and instructions.
* **Orchestrator Summary:** Combines all output steps to compile an executive overview.

### 🔍 Retrieval-Augmented Generation (RAG) Security Chat
1. The user asks a question in the RAG search box (e.g., *"What OpenSSL buffer overflow vulnerability exists?"*).
2. The query is sent to the Express backend, which calls the Python `embedding-service` to generate a 384-dimensional query vector.
3. The query vector is used to perform a cosine similarity search in the `qdrant` database.
4. The matching documents are retrieved, assembled into a security context block, and injected into the prompt.
5. Groq evaluates the context and responds with an accurate, hallucination-free explanation.

---

## 🔧 Deployment & Local Setup (Docker Compose)

### Prerequisites
* **Docker** & **Docker Compose** installed.
* **Groq API Key** (Get it from the Groq console).

### Environment Configuration
Create a `.env` file inside the `backend` folder:
```env
PORT=5000
MONGO_URI=mongodb://mongodb:27017/fullstackapp
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL=llama3-70b-8192
QDRANT_URL=http://qdrant:6333
QDRANT_COLLECTION=cve_knowledge_base_v2
EMBEDDING_URL=http://embedding-service:8000/embed
```

Configure `/frontend/.env` to point to the backend API:
```env
VITE_API_URL=http://localhost:5000
VITE_BACKEND_URL=http://localhost:5000
```

### Spin Up the Stack
Run the following command from the root of the project to build and start all five containers in detached mode:
```bash
docker-compose up -d --build
```

### Verification & Health Checking
You can inspect the running containers and logs using standard Docker CLI commands:
```bash
# Check running containers
docker-compose ps

# View backend logs
docker-compose logs -f backend

# Run the end-to-end RAG test script inside the backend container
docker-compose exec backend node test-end-to-end-rag.js
```

---

## 🧑‍💻 Resume and Portfolio Value

* **Multi-Container Microservice Orchestration:** Demonstrates production-level experience designing, configuring, and network-linking diverse container types (Node.js, Python FastAPI, MongoDB, Qdrant).
* **Enterprise DevSecOps & Security Tools:** Showcases hands-on knowledge in automations involving security incident monitoring, CVE assessment, risk classification, and patch remediation workflows.
* **On-Premise AI Deployment:** Proves you can deploy, containerize, and load machine learning models (HuggingFace transformers) locally in a private Python environment without relying on expensive SaaS APIs.
* **Persistent Volumes & Data Integrity:** Highlights knowledge of Docker volumes, ensuring database configurations (MongoDB, Qdrant) maintain state across container restarts.
