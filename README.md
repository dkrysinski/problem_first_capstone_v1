# RegTech AI - Multi-Framework Regulatory Compliance Assistant

AI-powered regulatory compliance system that analyzes business questions against EU regulatory frameworks (GDPR, NIS2, DORA, CER) using LangGraph orchestration and parallel processing.

## What It Does

- **Multi-Framework Analysis**: Simultaneously processes questions against 4 EU regulatory frameworks
- **Intelligent Classification**: Determines which regulations apply to business scenarios
- **Parallel Processing**: Concurrent framework analysis using LangGraph state machines
- **Synthesis**: Combines multi-framework insights into comprehensive compliance guidance

## Core Technologies

- **LangGraph**: State machine orchestration for multi-step regulatory analysis
- **LangChain**: RAG framework with Azure OpenAI integration (GPT-4)
- **FAISS**: Vector similarity search with framework-specific indices
- **Streamlit**: Authentication-protected web interface
- **Azure OpenAI**: Primary LLM via `gpt-4.1-maven-course` deployment

## LLM Optimization Techniques

### Structured Outputs
- **Pydantic Models**: Enforced response schemas for classification and analysis
- **Framework-Specific Prompts**: Tailored prompts for GDPR, NIS2, DORA, CER domains
- **Multi-Step Processing**: Classification → Analysis → Synthesis → Final Answer

### Parallel Processing
- **Concurrent Framework Analysis**: Multiple regulatory analyses execute simultaneously
- **State Graph Architecture**: LangGraph manages complex workflow dependencies
- **Smart Routing**: Dynamic execution based on classification results

### Document Retrieval Optimization
- **Framework-Specific Vector Stores**: Separate FAISS indices for targeted retrieval
- **Chunking Strategy**: RecursiveCharacterTextSplitter (1000 chars, 200 overlap)
- **Batch Processing Fallback**: Handles large documents with graceful degradation

## Guardrails & Safety

### Input Validation
- **Off-Topic Detection**: Classification model identifies non-regulatory questions
- **Rejection Node**: Dedicated workflow path for inappropriate queries
- **Structured Classification**: Confidence scoring for framework applicability

### Security Measures
- **Prompt Injection Protection**: Security safeguards in synthesis prompts
- **SHA-256 Authentication**: Hashed password protection for web access
- **Environment-Based Secrets**: No hardcoded credentials

### Content Filtering
- **Regulatory Scope Enforcement**: System only processes compliance-related queries
- **Source Attribution**: All responses include document citations
- **Context Validation**: UTF-8 encoding cleanup and content validation

## Evaluation & Monitoring

### Observability
- **Opik Integration**: AI performance tracing and monitoring (optional)
- **Debug Logging**: Comprehensive workflow tracking at each processing stage
- **State Tracking**: Full visibility into classification, routing, and synthesis decisions

### Quality Assurance
- **Citation Requirements**: All answers must include source document references
- **Multi-Framework Validation**: Cross-verification across applicable regulations
- **Synthesis Quality Control**: Consolidation process includes consistency checks

### Performance Metrics
- **Response Time Tracking**: Via Opik tracing when enabled
- **Document Retrieval Accuracy**: Vector similarity scoring
- **Classification Confidence**: Structured output includes certainty measures

## Architecture

```
Question → Classification → [GDPR|NIS2|DORA|CER Analysis] → Synthesis → Answer
                ↓
           Off-Topic → Rejection
```

**Key Components:**
- `agent.py`: LangGraph state machine with 8 processing nodes
- `prompts.py`: Framework-specific LLM prompts with structured outputs
- `app.py`: Streamlit interface with session management
- Vector stores: Framework-specific FAISS indices with embeddings

## Quick Start

```bash
# Setup
python -m venv venv && source venv/bin/activate
pip install langchain langchain-openai langgraph streamlit faiss-cpu

# Configure .env
AZURE_OPENAI_API_KEY=your_key
AZURE_OPENAI_ENDPOINT=your_endpoint

# Run
cd src/streamlit_app && streamlit run app.py
```