# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh && ./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Dependencies
```bash
# Install all dependencies
uv sync

# Install uv package manager (if needed)
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Environment Setup
Create `.env` file in root with:
```bash
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** for querying course materials. The application uses a multi-layer architecture with clear separation of concerns:

### Core Architecture Pattern
The system follows an **orchestrated RAG pattern** where `rag_system.py` acts as the central coordinator, delegating to specialized components:

- **Document Processing Pipeline**: `document_processor.py` → `vector_store.py` → ChromaDB
- **Query Processing Pipeline**: `app.py` → `rag_system.py` → `ai_generator.py` → Claude API
- **Tool-Based Search**: Claude dynamically decides whether to search course content or use general knowledge

### Key Components

**Backend (Python/FastAPI)**
- `app.py`: FastAPI server with API endpoints and static file serving
- `rag_system.py`: Central orchestrator coordinating all RAG operations
- `ai_generator.py`: Claude API integration with tool calling support
- `vector_store.py`: ChromaDB interface for semantic search
- `document_processor.py`: Text chunking and course parsing from structured format
- `search_tools.py`: Tool definitions for Claude's function calling
- `session_manager.py`: Conversation history management
- `models.py`: Pydantic data models (Course, Lesson, CourseChunk)
- `config.py`: Centralized configuration with environment variables

**Frontend (Vanilla Web)**
- `frontend/`: Simple HTML/CSS/JS interface that consumes the FastAPI backend

**Data Storage**
- `docs/`: Course materials in structured text format
- `chroma_db/`: ChromaDB vector database (created automatically)

### Document Format Convention

Course documents in `docs/` follow a specific structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: [lesson title]
Lesson Link: [lesson url]
[lesson content...]

Lesson 1: [lesson title]
[lesson content...]
```

### RAG Flow Architecture

1. **Document Ingestion**: Structured course files → chunked text → vector embeddings → ChromaDB
2. **Query Processing**: User query → RAG system → Claude API (with tools)
3. **Intelligent Routing**: Claude decides to search course materials OR use general knowledge
4. **Tool Execution**: If searching needed, semantic search in ChromaDB → relevant chunks
5. **Response Generation**: Claude synthesizes final answer from search results
6. **Source Tracking**: Citations automatically extracted and returned to frontend

### Configuration System

All system parameters are centralized in `config.py`:
- Model settings (Claude model, embedding model)
- Text processing (chunk size, overlap, max results)
- Database paths and session limits
- Environment variables loaded from `.env`

### Session Management

The system maintains conversation context through `session_manager.py`, enabling multi-turn conversations while keeping recent history for Claude's context window.

### Tool Integration

The application uses **Anthropic's tool calling** where Claude can dynamically invoke the `search_course_content` tool based on query analysis, making the system intelligent about when to search vs. when to use general knowledge.

## Access Points

- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs
- Main API endpoint: `POST /api/query`
- Course stats: `GET /api/courses`

## Important Notes

- The system automatically loads course documents from `docs/` on startup
- ChromaDB database is created automatically in `./chroma_db/`
- The application serves both API and frontend from the same FastAPI instance
- Course titles are used as unique identifiers throughout the system
- Text chunking uses sentence-based splitting with configurable overlap for better semantic coherence
- make sure to use uv to manage all dependencies
- use uv to run python files