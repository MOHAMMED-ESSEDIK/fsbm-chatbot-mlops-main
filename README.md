# 🎓 FSBM AI Assistant — End-to-End MLOps & DataOps Platform

> **An end-to-end AI assistant platform for FSBM, designed around modern MLOps, DataOps, observability, containerization, and local LLM inference.**

This project implements a modular architecture for an AI-powered university assistant capable of combining a conversational interface with an automated data pipeline, local LLM inference, persistent application data, vector-based knowledge retrieval, and full-stack observability.

The objective is not simply to build a chatbot.

The objective is to demonstrate how an **AI application can be engineered, deployed, monitored, and maintained as a complete production-oriented system**.

---

## 📌 Table of Contents

* [Overview](#-overview)
* [Project Goals](#-project-goals)
* [System Architecture](#-system-architecture)
* [Architecture Flow](#-architecture-flow)
* [Core Components](#-core-components)
* [DataOps Layer](#-dataops-layer)
* [LLM Engine](#-llm-engine)
* [Knowledge & Retrieval Layer](#-knowledge--retrieval-layer)
* [Application Backend](#-application-backend)
* [Frontend](#-frontend)
* [Observability](#-observability)
* [Infrastructure](#-infrastructure)
* [Repository Structure](#-repository-structure)
* [Data Flow](#-data-flow)
* [Configuration](#-configuration)
* [Running the Project](#-running-the-project)
* [Service Endpoints](#-service-endpoints)
* [Testing](#-testing)
* [Engineering Principles](#-engineering-principles)
* [Production Considerations](#-production-considerations)
* [Future Improvements](#-future-improvements)

---

# 🧠 Overview

The **FSBM AI Assistant** is designed as a university-oriented conversational AI platform.

Instead of treating the chatbot as a single Python application, the system separates the major responsibilities into independent services:

```text
                    ┌─────────────────────┐
                    │      FSBM User      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      Frontend       │
                    │      :3000          │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     FastAPI API     │
                    │      :8000          │
                    └───────┬─────┬───────┘
                            │     │
                ┌───────────┘     └────────────┐
                ▼                              ▼
       ┌─────────────────┐            ┌─────────────────┐
       │ Knowledge /     │            │     Ollama      │
       │ Vector Store    │            │  Local LLM      │
       └────────┬────────┘            └─────────────────┘
                │
                ▼
       ┌─────────────────┐
       │     DuckDB      │
       │ Pipeline Data   │
       └─────────────────┘

                 DATAOPS
                    │
                    ▼
             ┌─────────────┐
             │   Dagster   │
             │    :3001    │
             └─────────────┘

              OBSERVABILITY
                    │
       ┌────────────┼─────────────┐
       ▼            ▼             ▼
  Prometheus     Loki          Tempo
       │            │             │
       └────────────┼─────────────┘
                    ▼
                 Grafana
```

The architecture is containerized using Docker Compose, allowing the complete environment to be started as a group of cooperating services.

---

# 🎯 Project Goals

The project focuses on several engineering objectives.

### 1. AI Application

Provide an intelligent conversational interface for FSBM-related information.

### 2. DataOps

Build an organized and reproducible pipeline for collecting, transforming, validating, and preparing knowledge used by the assistant.

### 3. LLM Integration

Support local LLM inference through **Ollama**, reducing the need to depend exclusively on external inference APIs.

### 4. Knowledge Retrieval

Maintain a dedicated vector-store layer so that the assistant can work with institution-specific knowledge rather than relying only on the language model's pretrained knowledge.

### 5. Backend Engineering

Expose the AI functionality through a dedicated **FastAPI** backend.

### 6. Observability

Monitor application metrics, logs, and traces using the Grafana observability ecosystem.

### 7. Reproducible Infrastructure

Use Docker Compose and persistent Docker volumes to make the development environment reproducible.

---

# 🏗️ System Architecture

The platform is divided into several logical layers.

## Application Layer

```text
Frontend
   │
   ▼
FastAPI Backend
   │
   ├── LLM inference
   ├── Knowledge retrieval
   ├── application logic
   └── persistence
```

## Data Layer

```text
Sources
   │
   ▼
DataOps Pipeline
   │
   ▼
Processed Data
   │
   ├── DuckDB
   └── Vector Store
```

## Infrastructure Layer

```text
Docker Compose
      │
      ├── MariaDB
      ├── Ollama
      ├── Dagster
      ├── Backend
      └── Frontend
```

## Observability Layer

```text
Application / Containers
          │
     ┌────┼────┐
     ▼    ▼    ▼
 Metrics Logs Traces
     │    │    │
     ▼    ▼    ▼
Prometheus Loki Tempo
       \   |   /
          ▼
        Grafana
```

---

# 🔄 Architecture Flow

A typical user request follows this conceptual path:

```text
User
 │
 ▼
Web Interface
 │
 ▼
FastAPI
 │
 ├──────────────► Ollama
 │                    │
 │                    ▼
 │                 LLM
 │
 ├──────────────► Knowledge Layer
 │                    │
 │                    ▼
 │              Vector Retrieval
 │
 ▼
Response Construction
 │
 ▼
Frontend
 │
 ▼
User
```

At the same time, the application generates telemetry:

```text
FastAPI
 │
 ├── Metrics ──────► Prometheus
 │                       │
 │                       ▼
 │                    Grafana
 │
 ├── Logs ──────────► Promtail
 │                       │
 │                       ▼
 │                      Loki
 │
 └── Traces ────────► Tempo
```

This separation is important because **application functionality and operational observability are treated as different concerns**.

---

# 🧩 Core Components

| Component          | Role                                 |
| ------------------ | ------------------------------------ |
| **Frontend**       | User-facing chatbot interface        |
| **FastAPI**        | Backend API and application logic    |
| **Ollama**         | Local LLM inference                  |
| **MariaDB**        | Relational application database      |
| **Dagster**        | Data pipeline orchestration          |
| **DuckDB**         | Analytical/local pipeline storage    |
| **Vector Store**   | Retrieval-oriented knowledge storage |
| **Prometheus**     | Metrics collection                   |
| **Grafana**        | Monitoring and visualization         |
| **Loki**           | Log aggregation                      |
| **Promtail**       | Log collection                       |
| **Tempo**          | Distributed tracing                  |
| **cAdvisor**       | Container-level resource monitoring  |
| **Alertmanager**   | Alert routing                        |
| **Docker Compose** | Multi-service orchestration          |

The current Compose configuration defines these services and exposes their principal ports.

---

# 🔄 DataOps Layer

The `dataops/` directory contains the data-engineering side of the project.

The key idea is:

> **The chatbot should not depend on manually prepared data sitting inside application code.**

Instead, data should move through a reproducible pipeline.

Conceptually:

```text
Raw Sources
     │
     ▼
Ingestion
     │
     ▼
Cleaning
     │
     ▼
Transformation
     │
     ▼
Validation
     │
     ▼
Structured Dataset
     │
     ├──────────────► DuckDB
     │
     └──────────────► Embedding / Vector Pipeline
                              │
                              ▼
                         Vector Store
```

Dagster acts as the orchestration layer for this process.

The Docker Compose configuration builds Dagster from `./dataops` and exposes its UI through host port `3001`.

This creates a clear separation:

```text
DATA ENGINEERING
       ↓
Knowledge Preparation
       ↓
AI APPLICATION
```

---

# 🤖 LLM Engine

The `llm-engine/` directory contains the backend AI engine.

The backend is containerized separately from the frontend and is exposed on:

```text
localhost:8000
```

The container receives configuration for:

* Hugging Face authentication
* Ollama
* Hugging Face model caching
* DuckDB
* Vector storage
* OpenTelemetry tracing

The Compose configuration explicitly connects the backend to the Ollama service using:

```text
OLLAMA_BASE_URL=http://ollama:11434
```

and configures the backend to use:

```text
DUCKDB_PATH=/app/data/duckdb/fsbm.duckdb
VECTORSTORE_PATH=/app/vectorstore
```

It also configures OpenTelemetry to export telemetry toward Tempo.

---

# 🧠 Knowledge & Retrieval Layer

A university assistant needs access to information that a generic LLM may not know reliably.

For example:

```text
University documents
       │
       ▼
Data processing
       │
       ▼
Text chunks
       │
       ▼
Embeddings
       │
       ▼
Vector Store
       │
       ▼
Relevant Context
       │
       ▼
LLM
       │
       ▼
Grounded Response
```

This architecture allows the LLM to operate together with an institution-specific knowledge base.

The project maintains a persistent vector-store volume:

```text
vectorstore_cache
```

which is mounted into the backend at:

```text
/app/vectorstore
```

The design therefore separates:

**Knowledge retrieval**

from

**Language generation**.

That distinction is fundamental to building a useful institutional assistant.

---

# 🗄️ Database Layer

The application uses **MariaDB** for relational persistence.

The database is configured with:

```text
Database:
fsbm_assistant

Port:
3306
```

The Docker service is named:

```text
fsbm-mariadb
```

and stores its data in a persistent Docker volume:

```text
mariadb_data
```

This means restarting the containers does not automatically remove the database contents.

The frontend receives:

```text
DATABASE_URL=mysql://root:root@db:3306/fsbm_assistant
```

which allows it to communicate with MariaDB through the Docker network.

---

# 🦙 Ollama

Ollama provides the local LLM runtime.

The service:

```text
ollama/ollama:latest
```

runs on:

```text
localhost:11434
```

Models are stored in the persistent:

```text
ollama_models
```

volume.

The architecture therefore becomes:

```text
FastAPI
   │
   │ HTTP
   ▼
Ollama
   │
   ▼
Local LLM
```

This approach makes local model inference a first-class component of the platform.

---

# 🌐 Frontend

The `website/` directory contains the frontend application.

It is independently containerized and exposed through:

```text
http://localhost:3000
```

The frontend is configured with two important dependencies:

```text
DATABASE_URL
BACKEND_URL
```

Inside Docker:

```text
BACKEND_URL=http://backend:8000
```

Therefore the browser-facing application does not need to know the internal Docker networking details.

---

# 📊 Observability

One of the strongest architectural aspects of the project is that monitoring is treated as part of the system rather than an afterthought.

The project contains three complementary observability dimensions:

```text
             OBSERVABILITY
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
    Metrics      Logs       Traces
      │           │           │
      ▼           ▼           ▼
 Prometheus      Loki        Tempo
      │           │           │
      └───────────┼───────────┘
                  ▼
               Grafana
```

---

## 📈 Metrics — Prometheus

Prometheus collects numerical metrics from the application and infrastructure.

It runs on:

```text
http://localhost:9090
```

The project provides:

```text
monitoring/prometheus.yml
monitoring/prometheus_rules.yml
```

for Prometheus configuration and alerting rules.

Examples of metrics that can be useful include:

```text
request count
request latency
error rate
resource utilization
service availability
```

---

# 📉 Grafana

Grafana provides the visualization layer.

It runs on:

```text
http://localhost:3005
```

The project provisions Grafana data sources and dashboards from the repository:

```text
monitoring/grafana/provisioning/
monitoring/grafana/dashboards/
```

This makes dashboards part of the project configuration instead of relying entirely on manually created dashboards.

---

# 📝 Logging — Loki + Promtail

Application and container logs are handled using:

```text
Promtail
    │
    ▼
  Loki
    │
    ▼
 Grafana
```

Loki runs on:

```text
localhost:3100
```

Promtail collects logs from the Docker environment.

Relevant configuration files include:

```text
monitoring/loki-config.yml
monitoring/promtail-config.yml
```

This makes it possible to investigate problems without connecting manually to every container.

---

# 🔍 Distributed Tracing — Tempo

Tempo provides distributed tracing.

It runs on:

```text
localhost:3200
```

and exposes OTLP over:

```text
localhost:4317
```

The backend is configured to export OpenTelemetry data toward:

```text
http://tempo:4317
```

This makes it possible to follow a request through multiple components.

For example:

```text
Frontend
   │
   ▼
FastAPI
   │
   ├──► Vector Retrieval
   │
   └──► Ollama
           │
           ▼
          LLM
```

Tracing can help determine where latency is introduced.

---

# 🚨 Alerting

The architecture also includes **Alertmanager**.

It is exposed on:

```text
http://localhost:9093
```

Its configuration is stored in:

```text
monitoring/alertmanager.yml
```

Prometheus can evaluate alerting rules and pass triggered alerts to Alertmanager.

Conceptually:

```text
Metric
  │
  ▼
Prometheus
  │
  │ alert rule triggered
  ▼
Alertmanager
  │
  ▼
Notification
```

---

# 🐳 Container Monitoring — cAdvisor

cAdvisor provides container-level resource information.

It is exposed on:

```text
http://localhost:8080
```

The service has access to Docker and system information through read-only mounts.

This allows the monitoring stack to observe container resource behavior.

---

# 🗂️ Repository Structure

```text
fsbm-chatbot-mlops-main/
│
├── .github/
│   └── workflows/
│
├── dataops/
│   ├── Dockerfile
│   └── data pipeline components
│
├── deploy/
│   └── deployment configuration
│
├── docs/
│   └── project documentation
│
├── komodo/
│   └── deployment / infrastructure automation
│
├── llm-engine/
│   ├── Dockerfile
│   └── FastAPI / AI backend
│
├── monitoring/
│   ├── prometheus.yml
│   ├── prometheus_rules.yml
│   ├── alertmanager.yml
│   ├── loki-config.yml
│   ├── promtail-config.yml
│   ├── tempo-config.yml
│   └── grafana/
│       ├── provisioning/
│       └── dashboards/
│
├── tests/
│   └── automated tests
│
├── website/
│   └── frontend application
│
├── .dockerignore
├── .gitignore
├── docker-compose.yml
├── pyproject.toml
├── setup.ps1
└── README.md
```

The repository currently separates application, data, deployment, monitoring, testing, and frontend responsibilities rather than putting everything into a single directory.

---

# 🔁 Complete Data Flow

A simplified end-to-end view is:

```text
                  ┌───────────────────┐
                  │   FSBM Sources    │
                  └─────────┬─────────┘
                            │
                            ▼
                  ┌───────────────────┐
                  │     Dagster       │
                  │     DataOps       │
                  └─────────┬─────────┘
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
          ┌─────────────┐       ┌──────────────┐
          │   DuckDB    │       │ Vector Store │
          └─────────────┘       └──────┬───────┘
                                       │
                                       ▼
User ──► Frontend ──► FastAPI ──► Retrieval
                           │             │
                           │             ▼
                           │        Context
                           │             │
                           └─────────────┤
                                         ▼
                                      Ollama
                                         │
                                         ▼
                                       LLM
                                         │
                                         ▼
                                    AI Response
                                         │
                                         ▼
                                     Frontend
```

At the same time:

```text
Backend / Containers
       │
       ├────────► Prometheus ──────► Grafana
       │
       ├────────► Promtail ────────► Loki
       │
       └────────► OpenTelemetry ───► Tempo
```

This gives the system three major pipelines:

### Data Pipeline

```text
Source → Process → Store → Retrieve
```

### AI Pipeline

```text
Question → Retrieval → Context → LLM → Response
```

### Observability Pipeline

```text
Application → Metrics / Logs / Traces → Monitoring
```

---

# ⚙️ Configuration

The backend supports configuration through environment variables.

Important variables include:

```env
HF_TOKEN=your_huggingface_token

ENABLE_METRICS=true

OLLAMA_BASE_URL=http://ollama:11434

HF_HOME=/root/.cache/huggingface

DUCKDB_PATH=/app/data/duckdb/fsbm.duckdb

VECTORSTORE_PATH=/app/vectorstore

OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4317

OTEL_SERVICE_NAME=fsbm-fastapi-backend
```

The Docker Compose configuration passes these values to the backend service.

### Security Note

Credentials should **never be committed to Git**.

Use environment files or secret-management mechanisms for sensitive values.

---

# 🚀 Running the Project

## Prerequisites

Install:

* Docker
* Docker Compose
* Git

Clone the repository:

```bash
git clone https://github.com/MOHAMMED-ESSEDIK/fsbm-chatbot-mlops-main.git

cd fsbm-chatbot-mlops-main
```

---

## Environment Variables

Create the required environment configuration.

For example:

```bash
export HF_TOKEN="your_token"
```

If the frontend requires local environment configuration, configure:

```text
website/.env.local
```

according to the frontend application's requirements.

---

## Start the Platform

Run:

```bash
docker compose up --build
```

For detached execution:

```bash
docker compose up --build -d
```

Docker Compose builds the project services and starts the infrastructure as a single environment.

---

## Stop the Platform

```bash
docker compose down
```

To remove persistent volumes as well:

```bash
docker compose down -v
```

> ⚠️ Removing volumes deletes persisted database, model-cache, vector-store, and monitoring data.

---

# 🌐 Service Endpoints

After startup, the principal services are available at:

| Service         | URL                      |
| --------------- | ------------------------ |
| Frontend        | `http://localhost:3000`  |
| FastAPI Backend | `http://localhost:8000`  |
| Dagster         | `http://localhost:3001`  |
| Ollama          | `http://localhost:11434` |
| Prometheus      | `http://localhost:9090`  |
| Grafana         | `http://localhost:3005`  |
| cAdvisor        | `http://localhost:8080`  |
| Alertmanager    | `http://localhost:9093`  |
| Loki            | `http://localhost:3100`  |
| Tempo           | `http://localhost:3200`  |

These ports correspond to the current Docker Compose configuration.

---

# 🧪 Testing

The repository is configured for `pytest`.

The root `pyproject.toml` specifies:

```text
testpaths = ["tests"]
```

and adds:

```text
llm-engine
dataops
```

to the Python path for testing.

Run:

```bash
pytest
```

or:

```bash
python -m pytest
```

The project also configures Ruff with a maximum line length of 100 characters.

---

# 🧱 Engineering Principles

The project follows several important engineering principles.

## Separation of Concerns

Each major responsibility has its own component:

```text
Frontend
Backend
DataOps
LLM
Database
Monitoring
Deployment
Testing
```

This makes the architecture easier to evolve.

---

## Reproducibility

Docker Compose allows the infrastructure to be defined as code.

Instead of manually installing:

```text
MariaDB
Ollama
Prometheus
Grafana
Loki
Tempo
Dagster
```

the environment can be described and launched from the Compose configuration.

---

## Persistent State

Important state is stored through Docker volumes:

```text
mariadb_data
ollama_models
huggingface_cache
shared_pipeline_data
vectorstore_cache
prometheus_data
grafana_data
```

This separates persistent data from ephemeral containers.

---

## Observability by Design

Monitoring is integrated directly into the architecture.

The system does not only answer:

> "Does the chatbot work?"

It also aims to answer:

> "How is it performing?"

> "Where is latency coming from?"

> "What errors are occurring?"

> "What is happening inside the containers?"

> "Can we trace a request across services?"

---

# 🔐 Production Considerations

The current Compose configuration is primarily suitable as a development / demonstration environment.

Before production deployment, several areas should be hardened.

### Secrets

Current development configuration contains simple database credentials and Grafana credentials.

Production should use:

```text
Docker secrets
Vault
Cloud secret managers
Kubernetes secrets
```

rather than committing credentials.

### Database

MariaDB should use:

* non-root application users
* strong passwords
* backups
* restricted network access
* connection pooling

### LLM

Ollama should be evaluated according to:

* model size
* GPU availability
* inference latency
* concurrency
* memory requirements

### Observability

Production monitoring should include:

* service-level metrics
* latency percentiles
* error rates
* resource saturation
* alerting policies
* retention policies

### Deployment

For larger environments, Docker Compose can eventually be replaced or complemented by:

```text
Kubernetes
Helm
Cloud container platforms
```

depending on operational requirements.

---

# 🚧 Future Improvements

Potential extensions include:

## MLOps

* model/version tracking
* automated evaluation
* model quality gates
* automated retraining
* experiment tracking
* model registry

## DataOps

* data-quality checks
* schema validation
* pipeline lineage
* data versioning
* automated ingestion
* data freshness monitoring

## RAG

* improved chunking strategies
* embedding evaluation
* hybrid search
* reranking
* citation-aware answers
* retrieval evaluation

## CI/CD

A mature CI/CD pipeline could automatically perform:

```text
Git Push
   │
   ▼
Tests
   │
   ▼
Lint
   │
   ▼
Build Docker Images
   │
   ▼
Security Checks
   │
   ▼
Deploy
   │
   ▼
Smoke Tests
```

## Production Infrastructure

Possible future deployment architecture:

```text
                GitHub
                   │
                   ▼
                CI/CD
                   │
                   ▼
             Container Registry
                   │
                   ▼
              Kubernetes
                   │
       ┌───────────┼────────────┐
       ▼           ▼            ▼
    Backend      Frontend      DataOps
       │
       ├──────► LLM
       ├──────► Database
       └──────► Vector Store
                    │
                    ▼
             Observability
        ┌──────────┼───────────┐
        ▼          ▼           ▼
   Prometheus     Loki        Tempo
        └──────────┼───────────┘
                   ▼
                Grafana
```

---

# 📚 Project Philosophy

The central idea behind this project is that an AI assistant is not just an LLM.

A robust AI system consists of multiple engineering layers:

```text
                    AI PRODUCT
                        │
        ┌───────────────┼────────────────┐
        │               │                │
        ▼               ▼                ▼
    Interface       Intelligence       Data
        │               │                │
        │               ▼                ▼
        │              LLM            DataOps
        │               │                │
        └───────────────┼────────────────┘
                        │
                        ▼
                  Infrastructure
                        │
                        ▼
                  Observability
                        │
                        ▼
                    Operations
```

The project therefore combines:

**AI Engineering + Data Engineering + MLOps + DevOps + Observability**

into a single system.

---

# 👨‍💻 Author

**Mohammed Essedik**

Master's student in Big Data & Data Science
Faculty of Sciences Ben M'Sik — Hassan II University of Casablanca

GitHub:

https://github.com/MOHAMMED-ESSEDIK

---

# 📄 License

Add the project's chosen license here if/when one is included in the repository.
