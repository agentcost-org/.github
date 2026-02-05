# AgentCost

**Track, analyze, and optimize LLM costs in your AI applications.**

AgentCost is an open-source observability platform for LLM-powered applications. It automatically tracks every LLM call your AI agents make, calculates real-time costs, and provides actionable insights to reduce your AI infrastructure spending.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Python](https://img.shields.io/badge/python-3.10+-blue.svg)
![FastAPI](https://img.shields.io/badge/fastapi-0.109+-green.svg)

## The Problem

When building AI applications with LangChain, you have no visibility into:

- **Which agents** are costing you money
- **Which LLM calls** are happening under the hood
- **How to optimize** your prompts and model choices

AgentCost solves this with **zero code changes**.

## Features

- **Zero-Friction Integration**: Add 2 lines of code - your existing LangChain code works unchanged
- **Real-Time Tracking**: See every LLM call, token count, cost, and latency
- **Multi-Agent Support**: Track costs per agent in your multi-agent systems
- **Optimization Suggestions**: Get AI-powered recommendations to reduce costs
- **Analytics Dashboard**: Beautiful charts and trends (React-based)
- **Self-Hostable**: Run on your own infrastructure for data privacy

## Architecture

```
┌─────────────────────┐     ┌─────────────────────┐     ┌─────────────────────┐
│   Your LangChain    │     │   AgentCost SDK     │     │  AgentCost Backend  │
│    Application      │────▶│  (Python Package)   │────▶│    (FastAPI)        │
│                     │     │                     │     │                     │
│  - Agents           │     │  - Monkey patching  │     │  - Event ingestion  │
│  - Chains           │     │  - Token counting   │     │  - Analytics API    │
│  - LLMs             │     │  - Cost calculation │     │  - PostgreSQL       │
└─────────────────────┘     │  - Batching         │     └─────────────────────┘
                            └─────────────────────┘               │
                                                                  │
                                                    ┌─────────────▼─────────────┐
                                                    │   AgentCost Dashboard     │
                                                    │       (Next.js)           │
                                                    │                           │
                                                    │  - Cost overview          │
                                                    │  - Agent breakdown        │
                                                    │  - Time series charts     │
                                                    └───────────────────────────┘
```

## Quick Start

### 1. Install the SDK

```bash
pip install agentcost
# or from source:
pip install -e agentcost-sdk/
```

### 2. Add to Your Code (2 lines!)

```python
from agentcost import track_costs

# Initialize tracking
track_costs.init(
    api_key="your_api_key",
    project_id="my-project"
)

# Your existing LangChain code - works unchanged!
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")
response = llm.invoke("Hello, world!")  # ← Automatically tracked!
```

### 3. View Your Analytics

Start the backend and dashboard, then visit `http://localhost:3000` to see your costs.

## Project Structure

```
AgentCost/
├── agentcost-sdk/          # Python SDK (pip installable)
│   ├── agentcost/          # Core package
│   │   ├── tracker.py      # Main entry point
│   │   ├── interceptor.py  # LangChain monkey patching
│   │   ├── token_counter.py # tiktoken-based counting
│   │   ├── cost_calculator.py # Real-time cost calculation
│   │   ├── batcher.py      # Event batching (size + time)
│   │   └── http_client.py  # Backend communication
│   └── tests/              # SDK tests
│
├── agentcost-backend/      # FastAPI backend
│   ├── app/
│   │   ├── main.py         # Application entry point
│   │   ├── config.py       # Environment configuration
│   │   ├── database.py     # Async SQLAlchemy
│   │   ├── models/         # Pydantic & SQLAlchemy models
│   │   ├── routes/         # API endpoints
│   │   ├── services/       # Business logic
│   │   └── utils/          # Auth, helpers
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── agentcost-dashboard/    # Next.js frontend
│   └── src/
│       ├── app/            # Next.js app router
│       └── components/     # React components
│
├── demo_sdk.py             # SDK demo script
└── test_e2e.py             # End-to-end test
```

## Development Setup

### Prerequisites

- Python 3.10+
- Node.js 18+ (for dashboard)
- Docker (optional, for containerized deployment)

### 1. Clone & Setup

```bash
git clone https://github.com/yourusername/AgentCost.git
cd AgentCost

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install all dependencies
pip install -r requirements.txt
pip install -e agentcost-sdk/
pip install -r agentcost-backend/requirements.txt
```

### 2. Configure Environment

```bash
# Create .env file
cp agentcost-backend/.env.example agentcost-backend/.env

# Edit with your settings
# For development, defaults work fine
# For production, set SECRET_KEY and DATABASE_URL
```

### 3. Run the Backend

```bash
cd agentcost-backend
uvicorn app.main:app --reload --port 8000

# API available at http://localhost:8000
# Docs at http://localhost:8000/docs
```

### 4. Run the Dashboard

```bash
cd agentcost-dashboard
npm install
npm run dev

# Dashboard at http://localhost:3000
```

### 5. Run the Demo

```bash
# In another terminal
python demo_sdk.py
```

## How Costs Are Calculated

All costs are calculated in **real-time** using:

1. **Token Counting**: Uses `tiktoken` (OpenAI's tokenizer) for accurate counts
2. **Model Pricing**: Up-to-date pricing for **1,900+ models** from all major providers
3. **Formula**:
   ```
   cost = (input_tokens / 1000 × input_price) + (output_tokens / 1000 × output_price)
   ```

### Supported Providers (1,900+ Models)

AgentCost supports pricing for models from:

| Provider         | Examples                                     |
| ---------------- | -------------------------------------------- |
| **OpenAI**       | GPT-4, GPT-4-Turbo, GPT-4o, o1, o1-mini      |
| **Anthropic**    | Claude 3 Opus, Claude 3.5 Sonnet, Claude 4   |
| **Google**       | Gemini Pro, Gemini 1.5 Pro/Flash, Gemini 2.0 |
| **Mistral**      | Mistral Small, Medium, Large                 |
| **DeepSeek**     | DeepSeek Chat, DeepSeek Reasoner             |
| **Groq**         | Llama 3.x, Mixtral                           |
| **Cohere**       | Command, Command-R, Command-R+               |
| **Together AI**  | Meta Llama, Qwen, Phi                        |
| **AWS Bedrock**  | All Bedrock-hosted models                    |
| **Azure OpenAI** | All Azure-hosted OpenAI models               |
| **Perplexity**   | pplx-\* models                               |
| **And 30+ more** | Replicate, Fireworks, Anyscale, etc.         |

**Pricing is synced from [LiteLLM's pricing database](https://github.com/BerriAI/litellm)** which is continuously updated.

To sync pricing manually:

```bash
# Sync from LiteLLM (1600+ models)
curl -X POST http://localhost:8000/v1/pricing/sync/litellm

# Check sync status
curl http://localhost:8000/v1/pricing/sync/status
```

## Production Deployment

**Quick checklist:**

- [ ] Set `ENVIRONMENT=production`
- [ ] Set `SECRET_KEY` (use `python -c "import secrets; print(secrets.token_urlsafe(32))"`)
- [ ] Use PostgreSQL: `DATABASE_URL=postgresql+asyncpg://...`
- [ ] Configure CORS origins for your domain
- [ ] Set up HTTPS (nginx/Caddy)
- [ ] Remove `prototype.py` and test files

## Testing

```bash
# SDK tests
cd agentcost-sdk
pytest tests/ -v

# End-to-end test (requires backend running)
python test_e2e.py
```

## API Reference

### SDK

```python
# Initialize
track_costs.init(api_key="...", project_id="...", local_mode=False)

# Agent tagging
with track_costs.agent("my-agent"):
    llm.invoke("...")

# Add metadata
with track_costs.metadata(user_id="123", session_id="abc"):
    llm.invoke("...")

# Get local events (local_mode only)
events = track_costs.get_local_events()

# Manual flush
track_costs.flush()

# Shutdown
track_costs.shutdown()
```

### Backend API

| Endpoint                   | Method | Description                   |
| -------------------------- | ------ | ----------------------------- |
| `/v1/health`               | GET    | Health check                  |
| `/v1/projects`             | POST   | Create project                |
| `/v1/projects/{id}`        | GET    | Get project                   |
| `/v1/events/batch`         | POST   | Ingest events                 |
| `/v1/analytics/overview`   | GET    | Cost overview                 |
| `/v1/analytics/agents`     | GET    | Per-agent stats               |
| `/v1/analytics/models`     | GET    | Per-model stats               |
| `/v1/analytics/timeseries` | GET    | Time series data              |
| `/v1/optimizations`        | GET    | Cost optimization suggestions |

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests
5. Submit a pull request

## License

MIT License - see [LICENSE](./LICENSE) for details.

## Acknowledgments

- Built with [LangChain](https://langchain.com/)
- Token counting via [tiktoken](https://github.com/openai/tiktoken)
- Backend powered by [FastAPI](https://fastapi.tiangolo.com/)
- Dashboard built with [Next.js](https://nextjs.org/)
