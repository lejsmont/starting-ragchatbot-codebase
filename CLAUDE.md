# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a RAG (Retrieval-Augmented Generation) chatbot for querying course materials. It uses ChromaDB for vector storage, Anthropic's Claude for AI responses, and serves a web interface via FastAPI.

## Commands

**Always use `uv` to manage dependencies and run Python files (e.g., `uv run python script.py`) - do not use `pip` directly.**

```bash
# Install dependencies
uv sync

# Run the application (from project root)
./run.sh

# Or manually (from backend directory)
cd backend && uv run uvicorn app:app --reload --port 8000

# Access points
# Web UI: http://localhost:8000
# API docs: http://localhost:8000/docs
```

## Architecture

### Query Flow

```
Frontend (script.js)
    ↓ POST /api/query
FastAPI (app.py)
    ↓
RAGSystem (rag_system.py) - orchestrator
    ↓
AIGenerator (ai_generator.py) - calls Claude with tools
    ↓ (if Claude invokes tool)
CourseSearchTool (search_tools.py)
    ↓
VectorStore (vector_store.py) - ChromaDB semantic search
    ↓
Response returned with sources
```

### Backend Components (`backend/`)

| File | Role |
|------|------|
| `app.py` | FastAPI endpoints (`/api/query`, `/api/courses`), serves frontend |
| `rag_system.py` | Main orchestrator - connects all components |
| `ai_generator.py` | Claude API integration with tool calling |
| `vector_store.py` | ChromaDB interface with two collections: `course_catalog` (metadata) and `course_content` (chunks) |
| `document_processor.py` | Parses course docs, extracts metadata, chunks text |
| `search_tools.py` | `CourseSearchTool` implements the tool Claude can invoke; `ToolManager` handles registration/execution |
| `session_manager.py` | Tracks conversation history per session |
| `config.py` | Centralized settings (chunk size, model names, paths) |
| `models.py` | Pydantic models: `Course`, `Lesson`, `CourseChunk` |

### Frontend (`frontend/`)

Vanilla HTML/CSS/JS with Marked.js for markdown rendering. Communicates with backend via `/api/*` endpoints.

### Document Format

Course documents in `docs/` follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [name]

Lesson 0: [lesson title]
Lesson Link: [url]
[lesson content...]

Lesson 1: [lesson title]
...
```

### Key Configuration (`backend/config.py`)

- `CHUNK_SIZE`: 800 chars per chunk
- `CHUNK_OVERLAP`: 100 chars overlap between chunks
- `MAX_RESULTS`: 5 search results per query
- `MAX_HISTORY`: 2 messages of conversation context
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514

### Tool Calling Pattern

Claude receives the `search_course_content` tool definition. When invoked:
1. `AIGenerator._handle_tool_execution()` catches `stop_reason: "tool_use"`
2. `ToolManager.execute_tool()` routes to `CourseSearchTool.execute()`
3. Results formatted and sent back to Claude as `tool_result`
4. Claude generates final response incorporating search results

### Vector Store Collections

- **course_catalog**: Course metadata for name resolution (title → exact match)
- **course_content**: Chunked course text with embeddings for semantic search

## Environment

Requires `.env` file with:
```
ANTHROPIC_API_KEY=sk-ant-...
```
