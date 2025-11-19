# AEC Design Management RAG System

[![Python](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.109.0-009688.svg)](https://fastapi.tiangolo.com)
[![Docker](https://img.shields.io/badge/docker-ready-blue.svg)](https://www.docker.com/)
[![CI](https://github.com/hah23255/aec-rag-system/workflows/CI/badge.svg)](https://github.com/hah23255/aec-rag-system/actions)

A production-grade Retrieval-Augmented Generation (RAG) system for Architecture, Engineering, and Construction (AEC) design management, powered by GraphRAG and local LLMs.

> 📋 **For detailed codebase overview and statistics, see [CODEBASE_OVERVIEW.md](CODEBASE_OVERVIEW.md)**

## Features

### Core Capabilities
- **GraphRAG Architecture**: Relation-free graph construction using nano-graphrag or LinearRAG
- **Version Tracking**: Built-in support for drawing revisions with SUPERSEDES relationships
- **Impact Analysis**: Multi-hop reasoning to trace design change effects
- **Code Compliance**: Track building code requirements and component compliance
- **Document Processing**: Parse CAD files (DWG/DXF), PDFs, and scanned documents
- **Fully Local**: Zero API costs - runs entirely on local hardware

### Technical Stack
- **Embeddings**: nomic-embed-text-v1 (8K token context, 0.7GB VRAM)
- **LLM**: Llama-3.1-8B Q4 via Ollama (6GB VRAM)
- **GraphRAG**: nano-graphrag with NetworkX storage (scales to Neo4j)
- **Vector DB**: ChromaDB (embedded) or Milvus (production)
- **API**: FastAPI with async support
- **Deployment**: Docker Compose orchestration

## Quick Start

### Prerequisites
- Python 3.9+
- Docker & Docker Compose
- NVIDIA GPU with 16GB VRAM (RTX A5000 or equivalent)
- 16GB+ RAM
- Ubuntu 20.04+ or compatible Linux

### Installation

1. **Clone the repository**:
```bash
git clone https://github.com/hah23255/aec-rag-system.git
cd aec-rag-system
```

2. **Set up environment**:
```bash
# Copy environment template
cp .env.example .env

# Edit .env with your configuration
nano .env
```

3. **Start services with Docker Compose**:
```bash
# Start Ollama + API
docker-compose up -d

# Pull required models
docker exec aec-rag-ollama ollama pull nomic-embed-text
docker exec aec-rag-ollama ollama pull llama3.1:8b
```

4. **Verify installation**:
```bash
# Check API health
curl http://localhost:8000/api/v1/health

# View API documentation
open http://localhost:8000/api/docs
```

### Manual Installation (Development)

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Install Ollama separately
curl -fsSL https://ollama.com/install.sh | sh

# Pull models
ollama pull nomic-embed-text
ollama pull llama3.1:8b

# Run API
uvicorn src.api.main:app --reload --host 0.0.0.0 --port 8000
```

## Usage

### Upload a Document

```bash
# Upload a CAD file
curl -X POST "http://localhost:8000/api/v1/documents/upload" \
  -F "file=@/path/to/drawing.dxf" \
  -F "document_type=cad"

# Upload a PDF
curl -X POST "http://localhost:8000/api/v1/documents/upload" \
  -F "file=@/path/to/spec.pdf" \
  -F "document_type=pdf"
```

### Query the System

```bash
# Natural language query
curl -X POST "http://localhost:8000/api/v1/query" \
  -H "Content-Type: application/json" \
  -d '{"query": "What components are affected by changes to Drawing A-101?"}'

# Get version history
curl "http://localhost:8000/api/v1/versions/A-101"

# Impact analysis
curl "http://localhost:8000/api/v1/impact/component-id-123"
```

## Project Structure

```
aec-rag-system/
├── src/
│   ├── core/              # RAG core modules
│   │   ├── embeddings.py  # Embedding generation
│   │   ├── llm.py         # LLM interface
│   │   └── graphrag.py    # GraphRAG logic
│   ├── schema/            # AEC domain schema
│   │   └── aec_schema.py  # Entity & relationship definitions
│   ├── ingestion/         # Document processing
│   │   ├── cad_parser.py  # CAD file parsing
│   │   └── pdf_parser.py  # PDF parsing
│   ├── api/               # REST API
│   │   └── main.py        # FastAPI application
│   ├── retrieval/         # Query processing
│   └── utils/             # Utilities
├── tests/                 # Test suite
├── docs/                  # Documentation
├── scripts/               # Utility scripts
├── config/                # Configuration
├── deployment/            # Deployment configs
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── pyproject.toml
├── CODEBASE_OVERVIEW.md   # Detailed codebase documentation
└── README.md
```

## Development

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src --cov-report=html

# Run specific test suite
pytest tests/unit/
pytest tests/integration/
```

### Code Quality

```bash
# Format code
black src/ tests/

# Lint
ruff check src/ tests/

# Type check
mypy src/
```

### Adding New Features

1. Define entities/relationships in `src/schema/aec_schema.py`
2. Implement parsing logic in `src/ingestion/`
3. Add query capabilities in `src/retrieval/`
4. Expose via API in `src/api/main.py`
5. Write tests in `tests/`

## API Documentation

Interactive API documentation is available at:
- Swagger UI: `http://localhost:8000/api/docs`
- ReDoc: `http://localhost:8000/api/redoc`

### Key Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/health` | GET | Health check |
| `/api/v1/documents/upload` | POST | Upload document |
| `/api/v1/query` | POST | Natural language query |
| `/api/v1/versions/{drawing_id}` | GET | Version history |
| `/api/v1/impact/{entity_id}` | GET | Impact analysis |
| `/api/v1/graph/export` | GET | Export graph data |

## Architecture

### GraphRAG Flow

```
Document Upload → Parse → Extract Entities → Generate Embeddings
                                    ↓
                              Build Graph (NetworkX)
                                    ↓
Query → Embed → Retrieve Subgraph → LLM Reasoning → Response
```

### Resource Usage

| Component | VRAM | RAM | Notes |
|-----------|------|-----|-------|
| nomic-embed-text | 0.7 GB | 1 GB | Efficient embedding model |
| Llama-3.1-8B Q4 | 6.0 GB | 8 GB | Quantized for efficiency |
| API + Services | - | 2 GB | FastAPI, ChromaDB |
| **Total** | **7.7 GB** | **11 GB** | Fits RTX A5000 (16GB VRAM) |

## Deployment

### Docker Compose (Recommended)

```bash
# Production deployment
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Scale API instances
docker-compose up -d --scale api=3
```

### Kubernetes (Advanced)

```bash
# Apply manifests
kubectl apply -f deployment/k8s/

# Check status
kubectl get pods -n aec-rag
```

## Configuration

Key environment variables (see `.env.example`):

```bash
# Ollama
OLLAMA_HOST=http://localhost:11434
EMBEDDING_MODEL=nomic-embed-text
LLM_MODEL=llama3.1:8b

# API
API_HOST=0.0.0.0
API_PORT=8000
API_WORKERS=4

# Storage
GRAPH_BACKEND=networkx  # or neo4j
VECTOR_DB=chromadb      # or milvus
DATA_DIR=./data
```

## Troubleshooting

### Common Issues

**Ollama not responding**
```bash
# Check Ollama status
docker logs aec-rag-ollama

# Restart Ollama
docker-compose restart ollama
```

**Out of VRAM**
- Reduce batch sizes in `.env`
- Use smaller quantized models (Q3 instead of Q4)
- Close other GPU applications

**Slow queries**
- Check if models are loaded: `curl http://localhost:11434/api/tags`
- Enable embedding cache (default: enabled)
- Consider upgrading to Milvus for vector DB

## Performance

### Benchmarks (RTX A5000)

| Operation | Time | Throughput |
|-----------|------|------------|
| Embed 1K tokens | 50ms | 20K tokens/s |
| LLM generation (500 tokens) | 2-3s | ~200 tokens/s |
| CAD parsing (500KB DXF) | 1-2s | - |
| Graph query (3-hop) | 100ms | - |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development guidelines.

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file.

## Acknowledgments

- Based on [nano-graphrag](https://github.com/gusye1234/nano-graphrag) framework
- Inspired by [LinearRAG](https://github.com/NVIDIA/GenerativeAIExamples/tree/main/community/linear-rag) principles
- Built on [Ollama](https://ollama.com) for local LLM inference

## Support

- 📧 Email: support@example.com
- 🐛 Issues: [GitHub Issues](https://github.com/hah23255/aec-rag-system/issues)
- 💬 Discussions: [GitHub Discussions](https://github.com/hah23255/aec-rag-system/discussions)

---

**Status**: Production-ready v0.1.0 | **Last Updated**: November 2025
