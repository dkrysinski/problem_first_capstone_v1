# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a RegTech AI system built with Python that provides regulatory compliance guidance for EU frameworks (GDPR, NIS2, DORA, and CER). The system uses a multi-framework RAG (Retrieval-Augmented Generation) approach with LangGraph for orchestrated analysis across different regulatory domains.

## Architecture

### Core Components

- **Multi-Framework RAG Agent** (`src/langchain_modules/agent.py`): Central AI agent using LangGraph StateGraph that routes questions through multiple regulatory framework analyses
- **Streamlit Web Interface** (`src/streamlit_app/app.py`): Authentication-protected chat interface for user interactions
- **Framework-Specific Prompts** (`src/langchain_modules/prompts.py`): Specialized prompts for GDPR, NIS2, DORA, and CER analysis
- **Vector Stores**: FAISS-based document embeddings for each regulatory framework stored in `data/*_vector_store/`

### Processing Flow

1. **Classification Node**: Determines which regulatory frameworks apply to a business scenario using `CLASSIFICATION_PROMPT`
2. **Parallel Framework Analysis**: Concurrent analysis across applicable frameworks (GDPR, NIS2, DORA, CER) via dedicated nodes
3. **Synthesis Node**: Consolidates multi-framework results using `SYNTHESIS_PROMPT` into comprehensive compliance guidance
4. **Answer Generation**: Produces final structured regulatory advice with proper citations

### Key Technologies

- **LangChain**: Core RAG framework with Azure OpenAI integration
- **LangGraph**: State graph orchestration for multi-step regulatory analysis
- **FAISS**: Vector similarity search for document retrieval (with `allow_dangerous_deserialization=True`)
- **Streamlit**: Web interface with SHA-256 password authentication
- **Azure OpenAI**: GPT-4 model (`gpt-4.1-maven-course`) deployment via specific endpoint
- **Opik**: Optional tracing for performance monitoring (supports both local and cloud instances)

## Environment Setup

### Python Environment

- **Python Version**: 3.12.3
- **Virtual Environment**: `venv/` directory (must be activated)
- **Core Dependencies**: LangChain ecosystem, Streamlit, FAISS, dotenv

### Environment Variables

Configure `.env` file with:
```bash
# Azure OpenAI (primary LLM)
AZURE_OPENAI_API_KEY="your_azure_key"
AZURE_OPENAI_ENDPOINT="your_azure_endpoint"

# Authentication
STREAMLIT_USERNAME="username"
STREAMLIT_PASSWORD="password"

# Optional: Opik tracing
OPIK_USE_LOCAL=true  # or configure cloud instance
```

## Common Development Commands

### Running the Application

```bash
# Activate virtual environment
source venv/bin/activate

# Run Streamlit interface
cd src/streamlit_app
streamlit run app.py
```

### Testing the Agent Directly

```bash
# Activate environment first
source venv/bin/activate

# Test agent directly in Python (from project root)
python3 -c "
import sys
sys.path.append('src')
from langchain_modules.agent import AIAgent
agent = AIAgent()
print(agent.run('Test question about GDPR compliance'))
"
```

### Vector Store Management

Vector stores are automatically created from PDF documents in `data/` directory:
- `data/GDPR_Regulation.pdf` → `data/gdpr_vector_store/`
- `data/NIS2_Regulation.pdf` → `data/nis2_vector_store/`
- `data/DORA_Regulation.pdf` → `data/dora_vector_store/`
- `data/CER_Regulation.pdf` → `data/cer_vector_store/`

To regenerate vector stores, delete the corresponding `*_vector_store/` directory and restart the agent.

## Development Guidelines

### Agent Architecture Understanding

The agent follows a LangGraph state machine pattern with these critical nodes:
- `classification`: Uses structured output to determine applicable frameworks
- `gdpr_analysis`, `nis2_analysis`, `dora_analysis`, `cer_analysis`: Parallel framework-specific analysis
- `synthesis`: Combines all framework results into consolidated guidance
- `answering`: Final response generation with citations
- `rejection`: Handles off-topic questions with polite redirection

### Routing Logic

The `_route_to_applicable_frameworks()` method determines execution flow based on classification results. Key routing rules:
- Off-topic questions go to `rejection` node
- Framework flags (gdpr, nis2, dora, cer) determine which analysis nodes execute
- If no frameworks are selected, defaults to GDPR analysis
- All analysis results feed into synthesis before final answering

### Prompt Engineering

Each framework has specialized prompts in `prompts.py`:
- `CLASSIFICATION_PROMPT`: Determines regulatory applicability with confidence scores
- `RAG_PROMPT`: GDPR-specific analysis with structured response formatting  
- `NIS2_PROMPT`, `DORA_PROMPT`, `CER_PROMPT`: Framework-specific compliance guidance
- `SYNTHESIS_PROMPT`: Multi-framework consolidation with security safeguards

### Vector Store Implementation

- Uses RecursiveCharacterTextSplitter with 1000-character chunks and 200-character overlap
- Implements batch processing fallback for large documents
- Includes content validation and UTF-8 encoding cleanup
- Each framework maintains separate FAISS indices for targeted retrieval

### Security Considerations

- API keys and credentials are managed through `.env` (excluded from git)
- Streamlit app includes SHA-256 hashed password authentication
- Vector stores use `allow_dangerous_deserialization=True` (acceptable for controlled environment)
- Synthesis prompt includes security safeguards against prompt injection

### Error Handling Patterns

- Vector store creation with batch processing fallback if bulk operations fail
- Automatic retry for document processing errors with individual document isolation
- Graceful degradation when optional tracing (Opik) is unavailable
- Comprehensive error logging throughout the pipeline

### Performance Optimizations

- Vector stores are cached on disk to avoid regeneration
- Agent initialization includes `@st.cache_resource` for Streamlit efficiency
- Document processing includes validation to skip empty/invalid content
- Opik tracing is optional and can be disabled for faster processing

### Debugging and Monitoring

The agent includes extensive debug logging throughout the process:
- Classification debug logs show framework applicability reasoning
- Routing debug logs display which analysis nodes will execute
- Synthesis debug logs show document counts and framework combinations
- Opik integration provides comprehensive tracing when enabled